---
published: false
---
##Plugins: Maven vs Gradle

###¿Qué es un plugin?

Un plugin es aquella aplicación que, en un programa informático, añade una funcionalidad adicional o una nueva característica al software. En este caso puede nombrarse al plugin como un complemento o extensión de la herramienta [Maven](https://maven.apache.org/) o [Gradle](https://gradle.org/).

###¿Cuál es su utilidad?

En las herramientas de construcción de proyectos como son Maven o Gradle (además de ser gestores de dependencias) podemos extender o añadir funcionalidades en las diferentes fases de construcción, lo que nos permite realizar acciones como ejecutar tests, validar código, minificar archivos, compilar, empaquetar y arrancar la aplicación en un servidor...

###¿Cuándo se ejecutan?

En Maven, los _plugins_ se ejecutan en las diferentes fases en el momento de construcción del proyecto agrupadas en tres ciclos de vida diferentes: _clean_, _default_ y _site_. Sin entrar en más detalle (véase [Maven Lifecycle Reference](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Lifecycle_Reference)) tenemos fases dentro del ciclo de vida _default_ como son _process-resources_ (momento en el cual se copian los recursos en el directorio _target_), _compile_, _test_, _package_ (empaquetado del código compilado en formato JAR, por ejemplo) o _verify_ (comprobación de que los distintos controles como test unitarios, de integración o cuaquier otra comprobación se completa correctamente cuando se procesa el empaquetado cumpliendo así los criterios de calidad).

//Mostramos la configuración de fase de un plugin

En Gradle solo hay tres fases, de inicialización, de configuración y de ejecución. Sin embargo aquí se introducen las tareas o _Tasks_ las cuales ejecutan acciones. Estas tareas pueden ser ejecutadas según el orden deseado e introduciendo dependencias de orden entre ellas. Los plugins introducen nuevas tareas que son ejecutadas en el momento de ejecución y nuestras acciones pueden asociarse a la ejecución posterior o anterior a estas tareas.

//Mostramos la configuración dependiente de una tarea

###jshint (validando código JS)
