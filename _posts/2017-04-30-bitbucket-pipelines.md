---
layout: post
title: Bitbucket Pipelines
published: true
---
La integración continua ha llegado a los repositorios de [Bitbucket](https://bitbucket.org) y, aunque tiene sus limitaciones como el resto de servicios, se incluye gratuitamente. Por ahora, nos permitirá la ejecución ininterrumpida de _scripts_ durante un máximo de 2 horas donde contaremos con 4GB de RAM y 5GB de espacio en disco. La limitación mensual es de 500 minutos por usuario, y para los equipos de más de un usuario se multiplican estos minutos por cada uno, acumulando los minutos de ejecución (más información sobre las limitaciones del servicio [aquí](https://confluence.atlassian.com/bitbucket/limitations-of-bitbucket-pipelines-827106051.html)).

La ejecución se hará sobre un contenedor [_Docker_](https://www.docker.com), pudiendo seleccionar cualquier imagen pública o privada de [_Docker Hub_](https://hub.docker.com) o incluso nuestro propio contenedor con acceso público.

## ¿Cómo empezar?

Primero debemos habilitar [**Pipelines**](https://confluence.atlassian.com/bitbucket/bitbucket-pipelines-792496469.html) en el repositorio y a continuación se nos mostrarán las diferentes plantillas a seleccionar dependiendo del lenguaje utilizado. 

<center><img src="{{ site.baseurl }}/images/screen1.png" alt="Sección Pipelines en Bitbucket" width="450"></center>

El _script_ a ejecutar debe estar en el directorio raíz y llamarse **bitbucket-pipelines.yml**; este archivo indicará en formato _YAML_ la imagen de _Docker_, la rama a la que afecta, los pasos a ejecutar...

<center><img src="{{ site.baseurl }}/images/pipelines_yml_structure2.png" alt="Estructura del archivo bitbucket-pipelines.yml" width="450"></center>

## Ejecutando **Pipelines**

En el siguiente ejemplo vamos a configurar y ejecutar el script de **Pipelines** para un proyecto Java construido con Maven únicamente cuando hagamos _commits_ en la rama de _develop_ (siguiendo el modelo de ramificaciones de Vincent Driessen [git-flow](https://danielkummer.github.io/git-flow-cheatsheet/index.es_ES.html)) o se cree en el repositorio un _tag_ específico con la condición de que su nombre comience por _release_, en cuyo caso queremos realizar una acciones adicionales.

### Contenedor

En primer lugar indicaremos la imagen de _Docker_ a utilizar. Queremos disponer de Java 7 y Maven 3, por tanto indicaremos en el archivo de configuración la siguiente propiedad:

```yml
image: maven:3.3-jdk-7
```

### Rama

En segundo lugar vamos a indicar la rama de nuestro repositorio en la que se ejecutará **Pipelines** (bajo la sección _pipelines_ podemos indicar la sección _default_ que nos permitirá ejecutar el script en todas las ramas), que como hemos dicho antes, será solo cuando se revisen _commits_ en _develop_. Por tanto, en el archivo de configuración veremos los _steps_ dentro de la siguiente sección:

```yml
pipelines:
  branches:
    develop:
```
    
También debemos indicar los _steps_ a reproducir. Estos pasos se ejecutarán en el contenedor de _Docker_ el cual se iniciará de nuevo en cada ejecución. En el caso de ejemplo que se propone aquí queremos ejecutar el comando de _Maven_ llamado _verify_ en los _commits_ de la rama de _develop_. Por tanto, la sección quedará así:

```yml
pipelines:
  branches:
    develop:
	  - step:
        script:
          - mvn verify
```
          
Una vez generado y agregado el archivo _bitbucket-pipelines.yml_ en la rama _develop_ y hagamos el _commit_ (y posterior _push_) se ejecutará nuestro primer _Script_ de **Pipelines**. 

Es común que las dependencias utilizadas en nuestro proyecto se encuentren en repositorios privados y por tanto tengamos que indicar a _Maven_ la _url_ donde poder encontrar estos repositorios. Por tanto, desde _bash_, para construir nuestro proyecto Java, ejecutaríamos el comando de _Maven_ como **mvn -s settings.xml verify**. En el caso de **Pipelines** haremos exactamente lo mismo, incluyendo el archivo _settings.xml_ en el repositorio.

Es recomendable ejecutar _Maven_ con el parámetero -B que nos dará información del proceso _build_ y por tanto en caso de fallo podremos obtener un mayor contexto que nos ayudará a depurar los errores.

### En el caso de usar el _plugin surefire_

En el caso de usar el [_plugin surefire_](http://maven.apache.org/surefire/maven-surefire-plugin) que ejecuta los _test unitarios_ hay que tener en cuenta el uso de memoria que realiza. Si excedemos el uso de memoria, el proceso de **Pipelines** finalizará indicándonos que hemos excedido la memoria virtual asignada a nuestro proceso. 

<center><img src="{{ site.baseurl }}/images/ko-memory.png" alt="Sección Pipelines en Bitbucket" width="450"></center>

Puede provocarse este error en el caso de ejecutar _test unitarios_ que levanten contextos de _Spring_ para cada uno de los tests ejecutados; sin embargo solo sabremos que el proceso ha terminado por exceder la memoria, pero no quién o qué ha sido el causante del error. Para evitar que el motivo sea la ejecución de este _plugin_, en el archivo _pom.xml_ de nuestro proyecto definiremos la siguiente propiedad:

```xml
<properties>
  ...
  <argLine>-Xms512m -Xmx1024m -XX:MaxPermSize=512m</argLine>
</properties>
```

Este argumento es leído por el _plugin surefire_ y con esto nos aseguramos que la ejecución de los tests solo consuma como máximo 1GB comenzando a utilizar 512MB. Puesto que _Bitbucket_ nos ofrece 4GB de memoria podemos aumentar esta cifra si lo consideramos oportuno.

### Tag

Vamos ahora a indicar la otra sección (dentro de la sección _pipelines_), que indicará los _steps_ a realizar cuando se cree un _tag_ que comience por la palabra _release-*_ (nos ayudaremos de los _wildcards_):

```yml
  tags:
    release-*:
```
    
En este caso, cada vez que se haga una versión _release_ del código, queremos ejecutar un _scrip de bash_ el cual nos realiza de manera automática un _deploy_ en _Heroku_ por ejemplo. Solo tendríamos que incluir el _script_ dentro de nuestro código e indicar la ejecución del _step_:

```yml
  tags:
    release-*:
      script:
        - chmod +x scripts/release.bash
        - ./scripts/release.bash
```

En el caso anterior también contábamos con el comando _git_ y por lo tanto podemos hacer un _push_ desde el contenedor _Docker_ usando ssh con autenticación por claves. **Pipelines** nos permite definir variables de contexto que podemos utilizar en la ejecución de cada **Pipeline** (en el propio _YAML_ o incluso en el _script de bash_ del ejemplo anterior) y por tanto definir claves privadas o _tokens_ con los que acceder a servicios externos.

Finalmente, nuestro archivo _bitbucket-pipelines.yml_ ha quedado así:

```yml
image: maven:3.3-jdk-7

pipelines:
  branches:
    develop:
	  - step:
        script:
          - mvn -B -s settings.xml verify
          
  tags:
    release-*:
      script:
        - chmod +x scripts/release.bash
        - ./scripts/release.bash
```

## Ejecutando **Pipelines** en local

Si queremos ejecutar **Pipelines** en la máquina local solo tenemos que instalar _Docker_ y ejecutar desde el terminal el siguiente comando:

```bash
local$ docker run -it --volume=/Users/myUserName/code/localDebugRepo:/localDebugRepo --workdir="/localDebugRepo" --memory=4g --memory-swap=4g --entrypoint=/bin/bash maven:3.3-jdk-7
```

Esto creará un contenedor _Docker_ con las carácterísticas definidas y que actuará sobre el directorio indicado, que es el directorio del repositorio local. Solo nos quedará ejecutar los _steps_ dentro del contenedor; por tanto, ejecutaremos el comando siguiente:

```bash
docker$ mvn -B -s settings.xml verify
```

## Resultados

Por defecto nuestro **Pipeline** se ejecutará siempre que se lean nuevos _commits_ pero desde la web de _Bitbucket_ siempre podemos ejecutar de nuevo cada **Pipeline** manualmente.

Al ejecutarse nuestro **Pipeline** es posible obtener dos estados: correcto o incorrecto. Si la ejecución de nuestro _script_ provoca un mensaje por la salida de error, **Pipelines** acabará con el estado incorrecto indicando que la ejecución falló (y además lo notificará enviando un mail al propietario del _commit_ que causó el error):

<center><img src="{{ site.baseurl }}/images/ko.png" alt="Estructura del archivo bitbucket-pipelines.yml" width="450"></center>

En el caso contrario nos marcará que el estado es correcto:

<center><img src="{{ site.baseurl }}/images/ok.png" alt="Estructura del archivo bitbucket-pipelines.yml" width="450"></center>
