---
published: false
---
##Plugins: Maven vs Gradle

###¿Qué es un plugin?

Maven: build plugins (dejamos de lado los report plugins)
Gradle: binary plugins (script plugins no los tocamos)

Un plugin es aquella aplicación que, en un programa informático, añade una funcionalidad adicional o una nueva característica al software. En este caso puede nombrarse al plugin como un complemento o extensión de la herramienta [Maven](https://maven.apache.org/) o [Gradle](https://gradle.org/).

###¿Cuál es su utilidad?

En las herramientas de construcción de proyectos como son Maven o Gradle (además de ser gestores de dependencias) podemos extender o añadir funcionalidades en las diferentes fases de construcción, lo que nos permite realizar acciones como ejecutar tests, validar código, minificar archivos, compilar, empaquetar y arrancar la aplicación en un servidor... Estas acciones de validación, preprocesamiento o postprocesamiento que podemos hacer externamente desde _scripts_ quedan integrados con la herramienta de construcción y por tanto tenemos un único punto desde donde ejecutar todas estas tareas. Además se gestionan como si de una dependencia se tratase, lo que nos permite importar y ejecutar los plugins que consideremos desde nuestro proyecto usando el gestor de dependencias, como por ejemplo, Nexus.

###¿Cuándo se ejecutan?

En Maven, los _plugins_ se ejecutan en las diferentes fases de construcción del proyecto agrupadas en tres ciclos de vida diferentes: _clean_, _default_ y _site_. Sin entrar en más detalle (véase [Maven Lifecycle Reference](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Lifecycle_Reference)) tenemos fases dentro del ciclo de vida _default_ como son _process-resources_ (momento en el cual se copian los recursos en el directorio _target_), _compile_, _test_, _package_ (empaquetado del código compilado en formato JAR, por ejemplo) o _verify_ (comprobación de que los distintos controles como test unitarios, de integración o cuaquier otra comprobación se completa correctamente cuando se procesa el empaquetado cumpliendo así los criterios de calidad).

```xml
<myxml>
   <someElement />  
</myxml>
```

En Gradle solo hay tres fases, de inicialización, de configuración y de ejecución. Sin embargo aquí se introducen las tareas o _Tasks_ las cuales ejecutan acciones. Estas tareas pueden ser ejecutadas según el orden deseado e introduciendo dependencias de orden entre ellas. Los plugins introducen nuevas tareas que son ejecutadas en el momento de ejecución y nuestras acciones pueden asociarse a la ejecución posterior o anterior a estas tareas.

//Mostramos la configuración dependiente de una tarea

###¿Cómo los configuramos?

###¿Cómo los ejecutamos?

###java-exec

###jshint (validando código JS)

###yuicompressor (minificando código JS)

###jetty (servidor web) 

###¿Cómo los desarrollamos?
