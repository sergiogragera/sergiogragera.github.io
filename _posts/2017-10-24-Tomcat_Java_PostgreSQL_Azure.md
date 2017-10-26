---
layout: post
title: >-
  Desplegando aplicaciones web con Java en Tomcat + Base de datos PostgreSQL en
  Azure
published: true
---
Azure es un conjunto integral de servicios en la nube que permite crear, implementar y administrar aplicaciones a través de la red global de centros de datos de Microsoft.

[Aquí](https://azure.microsoft.com/es-es/pricing/) se puede ver el listado de precios de sus múltiples servicios y además cuenta con una [calculadora de precios](https://azure.microsoft.com/en-us/pricing/calculator/) como los otros grandes servicios de _cloud_ que existen actualmente (no da tanto miedo como la de AWS por ahora).

### Creando la base de datos

En este despliegue vamos a usar la [BBDD de Azure para servidor PostgreSQL](https://docs.microsoft.com/es-es/azure/postgresql/quickstart-create-server-database-portal) que se puede encontrar en el menú de bases de datos llamado **SQL Database**. Necesitaremos indicar el nombre del servidor, grupo de recursos (su función es agruparnos los grupos de recursos para poder gestionarlos de manera común y nos facilitará la vida organizando estos recursos cuando disponemos de múltiples servicios de aplicaciones y clientes), etc.

El tipo de rendimiento que hemos seleccionado en el ejemplo es el mínimo para este tipo de BBDD (plan de tarifa _Básico_, sin embargo, el plan Estándar es la opción ideal para aplicaciones en la nube que necesiten de un valor de E/S por segundo garantizado con un alto rendimiento). Es importante comentar que por ahora no se permite el escalado vertical entre planes de tarifas y solo se nos permite el escalado horizontal dentro de un mismo plan. Por tanto, este nos permitirá escalar de 50 unidades de proceso (medida combinada de recursos de CPU y memoria: 50 equivalen a medio núcleo) a 100. Para poder escalar de 100 a 200, 300, etc. debemos cambiar el plan de rendimiento y por tanto crear un nuevo servidor de BBDD:

Básico, 50 unidades de proceso, 50 GB, sin garantía de IOPS

Y dependiendo del nombre seleccionado para el servidor tendremos la url de conexión siguiente:

jdbc:postgresql://gocreuniones2.postgres.database.azure.com:5432/postgres?user=\[user@server\]&password=\[password\]&ssl=true

Si el servidor tiene el servicio de SSL habilitado necesitaremos descargar el certificado para usarlo en las conexiones que realicemos al servidor de base de datos creado. Para esto accederemos a la página de documentación sobre la [conexión con SSL](https://docs.microsoft.com/es-es/azure/postgresql/concepts-ssl-connection-security) y descargaremos el certificado desde el [enlace proporcionado](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt). Tras esto, debemos decodificar el archivo descargado almacenándolo en un archivo local que utilizaremos en las conexiones:

```openssl x509 -inform DER -in BaltimoreCyberTrustRoot.crt -text -out root.crt```

Además de estos aspectos de seguridad para la conexión (SSL es opcional pero su uso es esencial si queremos tener un cierto nivel de seguridad) es necesario configurar el **firewall** del servidor para permitir conectar desde otras máquina (si es que necesitamos conectar desde por ejemplo de una máquina cliente para crear bases de datos como se explica en el párrafo siguiente). Para ello, desde el propio menú del servidor de BBDD accederemos a la entrada de _Seguridad de la conexión_ y aquí veremos un apartado denominado _Reglas de firewall_ donde nosotros indicaremos un rango de IPS permitidas: por ejemplo nuestra dirección IP de la máquina donde estamos trabajando (repitiendo la IP lo que definimos es el acceso para una única IP y no para un rango de IPS).

```all -- 0.0.0.0 -- 255.255.255.255```

La única base de datos existente al crear el servidor es la llamada **postgres**, pero podremos conectar al servidor de base de datos para crear otras. En este caso, podemos conectar desde nuestro sistema (necesitaremos tener instalado el cliente de PostgreSQL) o bien desde la consola bash que proporciona el propio portal de manera online:

```psql "sslmode=verify-ca sslrootcert=root.crt host=[SERVER_NAME].postgres.database.azure.com dbname=postgres user=[USER_NAME]"```

Una vez conectados al servidor podremos crear cualquier otra base de datos (además podremos crear las tablas y asignar los roles). En este punto tendremos ya nuestra BBDD creada. Veamos entonces como desplegar nuestra aplicación web sobre el servidor de aplicaciones _App service_.

### Creando el servidor de aplicaciones

Seleccionamos ahora la entrada de menú _App Services_ donde agregaremos nuestro servidor o servicio de aplicaciones. Debemos definir el nombre de la aplicación, grupo de recurso al que asociaremos el servicio, plataforma, en este caso seleccionamos Windows, y lo más importante, el plan del servicio de aplicaciones o _Plan de App Service_ que indica las características del entorno donde se desplegará nuestro servicio. Aquí elegiremos el plan más barato de los estándar que son los que incluyen la posibilidad de realizar copias de seguridad:

<center><img src="{{ site.baseurl }}/images/precios.png" alt="Planes de App Services"></center>

Tras la selección de un _Plan de App Services_ realizaremos la configuración de nuestro servicio. En nuestro caso seleccionaremos la versión de Java 8 y el contenedor web, que al haber seleccionado Java, nos permite elegir entre diferentes versiones de Tomcat y Jetty. En el ejemplo elegiremos Tomcat 8.0 en su versión más reciente (dejaremos que sea Azure quien se encargue de actualizar la versión de Tomcat 8 en sucesivas versiones y correcciones). También elegiremos la plataforma de 64 bits.

<center><img src="{{ site.baseurl }}/images/configuracion_basica.png" alt="Configuración básica" width="450"></center>

Siempre podremos volver a editar esta configuración pero inicialmente podemos definir como propiedades de conexión la url de BBDD para que Azure nos permita hacer copias de seguridad del servicio de aplicaiones junto con los datos de la BBDD. Para esto es necesario definir la cadena de conexión (notése que en la imagen siguiente la url del jdbc se oculta por seguridad pero deberemos indicar la cadena de conexión con usuario y contraseña).

Esta propiedad nos servirá, además de como veremos en el siguiente párrafo para realizar copias de seguridad de la aplicación junto con la BBDD, también para tener disponibles como variables de entorno en tiempo de ejecución estos valores de conexión o credenciales que utilizará nuestra aplicación.

En el ejemplo llamamos a la variable GOC_JDBC y según la especificación para variables de conexión PostgreSQL se definirá en el entorno como **CUSTOMCONNSTR_GOC_JDBC**, siendo CUSTOMCONNSTR_ el prefijo para la conexión de PostgreSQL. Los prefijos de variable de entorno son los siguientes:

- SQL Server: SQLCONNSTR_
- MySQL: MYSQLCONNSTR_
- Base de datos SQL: SQLAZURECONNSTR_
- Personalizado: CUSTOMCONNSTR_

<center><img src="{{ site.baseurl }}/images/configuracion_conexion.png" alt="Configuración de la conexión de BBDD para back up" width="450"></center>

Una vez finalizada la configuración seleccionaremos la entrada de menú de _Copias de seguridad_. Aquí debemos seleccionar un _storage_ (el plan seleccionado anteriormente nos permite crear uno de 50 GB) donde ir guardando las copias y podemos programar la hora y fecha en las que se haran estas mismas. Además incluiremos la copia de la BBDD indicando la propiedad de la cadena de conexión que habíamos definido en el apartado anterior.

<center><img src="{{ site.baseurl }}/images/configuracion_backup.png" alt="Configuración de back up" width="450"></center>

Finalmente activaremos el registro de _logs_ desde la sección _Registros de diagnósticos_ donde podemos definir el _Registro de la aplicación_ en el sistema de archivos (esta opción se deshabilitará a las 12 horas) eligiendo también el nivel de log (detallado equivale al nivel DEBUG). Podremos acceder a estos archivos desde el menú _Secuencias de registro_.

<center><img src="{{ site.baseurl }}/images/logs.png" alt="Logs" width="450"></center>

Otra herramienta a tener en cuenta es Kudu que nos permite lanzar un terminal en el directorio donde se encuentra alojado nuestro servicio de aplicaciones. Aquí podemos navegar hasta el directorio home\site\wwwroot\webapps donde encontraremos nuestras aplicaciones desplegadas así como podremos hacer tail sobre el archivo de _log_ que estemos escribiendo desde nuestra aplicación, por ejemplo con _logback_.

<center><img src="{{ site.baseurl }}/images/kudu_tail.png" alt="Herramienta Kudu online" width="450"></center>

### Desplegando nuestra aplicación vía FTP

Vamos a explicar ahora como hacer el despliegue de nuestra aplicación web Java. Para esto hemos seleccionado la opción de subir el archivo WAR en el directorio creado por Azure para el despliegue tras elegir como contenedor Tomcat: el directorio home\site\wwwroot\webapps.

Para poder conectar por FTP a nuestro servidor de aplicaciones debemos configurar primero las credenciales de acceso. Accedemos por tanto al apartado _Credenciales de implementación_ donde definiremos el usuario y contraseña a utilizar en la conexión FTP.

En la sección de menú _Introducción_ se nos muestra toda la información relativa a la conexión:

```
FTP/Nombre de usuario de implementación
test\test

Nombre de host de FTP
ftp://waws-prod-am2-xxx.ftp.azurewebsites.windows.net

Nombre de host de FTPS
ftps://waws-prod-am2-xxx.ftp.azurewebsites.windows.net
```

Finalmente solo tenemos que conectar al servidor FTP y copiar nuestro archivo WAR en el directorio _webapps_ y entonces será cuando _App Services_ (concretamente el contenedor Tomcat) desplegará automaticamente nuestra aplicación cada vez que subamos un archivo.

### Calculando nuestros costes

El coste que calcularemos será mensual (744 horas) aunque la tarificación que hacen por defecto es por horas. Los recursos activos (estén arrancados o parados) tendrán el coste que aquí presentamos y únicamente tenemos la opción de eliminar los recursos para que no reporten ningún coste. 

Para recuperar un recurso, desde el menú de recursos y accediendo al nombre del grupo al que pertenecía tenemos una opción de menú llamado _Implementaciones_. Aquí se mostrarán las impementaciones que han existido en nuestro grupo de recursos y podemos recuperarlas facilmente seleccionando las mismas y pulsando sobre _Volver a implementar_.

La transferencia de datos tiene un coste a partir de los primeros 5 GB de 0.074€ por cada _gigabyte_ en la zona 1 (que incluye a Europa y Estados Unidos entre otras).

<center><img src="{{ site.baseurl }}/images/calculator.png" alt="Calculadora de precios" width="450"></center>

### Referencias

1. [Kit de herramientas de Azure para IntelliJ](https://docs.microsoft.com/es-es/java/azure/intellij/azure-toolkit-for-intellij). Uno de los puntos más interesantes de este plugin es poder depurar remotamente nuestra aplicación; sin embargo por problemas con las últimas versiones del _plugin_ han deshabilitado [esta opción](https://docs.microsoft.com/es-es/java/azure/intellij/azure-toolkit-for-intellij-whats-new) en la versión del 14 de abril de 2017 sin especificar cuándo estará nuevamente disponible.

1. Un post con la [comparativa de precios entre AWS, Azure y Google Cloud](https://www.simform.com/compute-pricing-comparison-aws-azure-googlecloud/) muy detallado.
