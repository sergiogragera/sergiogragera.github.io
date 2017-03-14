---
published: false
---
##Plugins: Maven vs Gradle

###¿Qué es un plugin?

Un plugin es aquella aplicación que, en un programa informático, añade una funcionalidad adicional o una nueva característica al software. En este caso puede nombrarse al plugin como un complemento o extensión de la herramienta [Maven](https://maven.apache.org/) o [Gradle](https://gradle.org/).

###¿Cuál es su utilidad?

En las herramientas de construcción de proyectos como son Maven o Gradle (además de ser gestores de dependencias) podemos extender o añadir funcionalidades en las diferentes fases de construcción, lo que nos permite realizar acciones como ejecutar tests, validar código, minificar archivos, compilar, empaquetar y arrancar la aplicación en un servidor...

En Maven, existen diferentes fases en el momento de construcción del proyecto divididas en tres ciclos de vida diferentes: clean, default y site.

- Initialization: se determina que proyectos van a formar parte de la construcción.
- Configuration: se ejecutan los scripts de construcción (buildScripts) de todos los proyectos.
- Execution: se determina el subconjunto de tareas creadas y configuradas y se ejecutan.

###jshint (validando código JS)
