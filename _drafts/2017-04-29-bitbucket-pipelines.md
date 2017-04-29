---
published: false
---
## Bitbucket Pipelines

La integración continua ha llegado a los repositorios de Bitbucket, y todavía, de manera gratuita; aunque si bien es cierto, tiene sus limitaciones como el resto de servicios. Por ahora, nos permitirá la ejecución ininterrumpida de _scripts_ durante 2 horas donde contaremos con 4GB de RAM y 5GB de espacio en disco. La limitación mensual es de 500 minutos por usuario, y para los equipos de más de un usuario se multiplican estos minutos por cada uno, acumulando los minutos de ejecución.

La ejecución se hará sobre un contenedor _docker_, pudiendo seleccionar cuaquier imagen pública o privada de _Docker Hub_ o incluso nuestro propio contenedore hospedado con acceso público.

## ¿Cómo empezar?

Primero debemos habilitar **Pipelines** en el repositorio y a continuación se nos mostrarán las diferentes plantillas a seleccionar dependiendo del lenguaje utilizado. El _script_ a ejecutar debe estar en el directorio raíz y llamarse **bitbucket-pipelines.yml**; este archivo indicará en formato _YAML_ la imagen de _docker_, la rama a la que afecta, los pasos a ejecutar...

[Imagen](https://confluence.atlassian.com/bitbucket/files/792298910/859457211/6/1490664866213/pipelines_yml_structure2.png)

## Ejecutando _Pipelines_

En el siguiente ejemplo vamos a configurar y ejecutar el script de _Pipelines_ para un proyecto Java construido con Maven únicamente cuando hagamos _commits_ en la rama de _develop_ (siguiendo el modelo de ramificaciones de Vincent Driessen [git-flow](https://danielkummer.github.io/git-flow-cheatsheet/index.es_ES.html)) o se cree en el repositorio un _tag_ específico como puede ser que comience por _release_, en cuyo caso queremos realizar una acciones adicionales.

### Imagen de Docker

En primer lugar indicaremos la imagen de _docker_ a utilizar: queremos disponer de Java 7 y Maven 3; por tanto indicaremos en el archivo de configuración la propiedad:

```yml
image: maven:3.3-jdk-7**
```

### Rama

En segundo lugar vamos a indicar la rama en la que se ejecutará **Pipelines**, que como hemos visto antes será en _develop_. Por tanto, en el archivo de configuración veremos los _steps_ dentro de la siguiente sección:

```yml
pipelines:
  branches:
    develop:
```
    
También debemos indicar los _steps_ a reproducir. Estos pasos se ejecutarán en el contenedor de _Docker_ el cual se iniciará de nuevo en cada ejecución. En el caso de ejemplo que se propone aquí queremos ejecutar el comando de _Maven_ _verify_ en los _commits_ de la rama de _develop_. Por tanto, la sección quedará así:

```yml
pipelines:
  branches:
    develop:
	  - step:
        script:
          - mvn verify
```
          
Una vez generado el archivo _bitbucket-pipelines.yml_ en la rama _develop_ y hagamos el _commit_ se ejecutará nuestro primer _Script_ de **Pipelines**. 

Es común que las dependencias utilizadas en nuestro proyecto se encuentren en repositorios privados y por tanto tengamos que indicar a _Maven_ la _url_ donde poder encontrar estos repositorios. Por tanto, desde _bash_ ejecutaríamos el comando como **mvn -s settings.xml verify**. En el caso de **Pipelines** haremos exactamente lo mismo, incluyendo el archivo _settings.xml_ en el repositorio.

Es recomendable ejecutar _Maven_ con el parámetero -B que nos dará información del proceso _build_ y por tanto en caso de fallo podremos obtener un mayor contexto que nos ayudará a depurar los errores.

### En el caso de usar el plugin surefire

En el caso de usar el _plugin surefire_ que ejecuta los _test unitarios_ hay que tener en cuenta el uso de memoria que realiza. Si excedemos el uso de memoria, el proceso finalizará indicándonos que hemos excedido la memoria virtual asignada a nuestro proceso. En el caso de ejecutar _test unitarios_ que levanten contextos de _Spring_ para cada uno de ellos puede conllevar a este error; sin embargo solo sabremos que el proceso ha terminado por exceder la memoria pero no quién o qué ha sido el causante. Para evitar que el causante sea la ejecución de este _plugin_, en el archivo _pom.xml_ definiremos la siguiente propiedad:

```xml
<properties>
  ...
  <argLine>-Xms512m -Xmx1024m -XX:MaxPermSize=512m</argLine>
</properties>
```

Con esto nos aseguramos que la ejecución de los test solo consuma como máximo 1GB comenzando a utilizar 512MB. Puesto que _Bitbucket_ nos ofrece 4GB de memoria podemos aumentar esta cifra si lo consideramos oportuno.

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

En el caso anterior también contábamos con el comando _git_ y por lo tanto podemos hacer un _push_ desde el contenedor _Docker_ usando ssh. **Pipelines** nos permite definir variables de contexto que podemos utilizar en la ejecución de cada **Pipeline** (en el propio YAML o incluso en el _script de bash_ del ejemplo anterior) y por tanto definir claves privadas o _tokens_ con los que acceder a servicios externos.

Finalmente, nuestro archivo _bitbucket-pipelines.yml_ ha quedado así:

```yml
image: maven:3.3-jdk-7**

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