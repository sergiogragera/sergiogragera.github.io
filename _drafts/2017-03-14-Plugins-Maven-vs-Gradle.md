---
published: false
---
##Plugins: Maven vs Gradle

###¿Qué es un plugin?

Un plugin es aquella aplicación que, en un programa informático, añade una funcionalidad adicional o una nueva característica al software. En este caso puede nombrarse al plugin como un complemento o extensión de la herramienta [Maven](https://maven.apache.org/) o [Gradle](https://gradle.org/).

###¿Cuál es su utilidad?

En las herramientas de construcción de proyectos como son Maven o Gradle (además de ser gestores de dependencias) podemos extender o añadir funcionalidades en las diferentes fases de construcción que nos permiten desde ejecutar tests, validar código, minificar archivos, _deployar_ y arrancar la aplicación en un servidor...

- validate: valida que el proyecto es correcto.
- compile: compila el código fuente del proyecto.
- test: ejecuta los test sobre el código compilado con un framework de test unitarios.
- package: empaqueta el código compilado en un formato válido como es JAR.
- verify: comprueba cualquier resultado de los tests de integración.
- install: instala el paquete en el repositorio local.
- deploy: instala el paquete final en el repositorio remoto.


- Initialization: se determina que proyectos van a formar parte de la construcción.
- Configuration: se ejecutan los scripts de construcción (buildScripts) de todos los proyectos.
- Execution: se determina el subconjunto de tareas creadas y configuradas y se ejecutan.

###jshint (validando código JS)
