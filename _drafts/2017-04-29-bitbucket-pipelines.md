---
published: false
---
## Bitbucket Pipelines

Enter text in [Markdown](http://daringfireball.net/projects/markdown/). Use the toolbar above, or click the **?** button for formatting help.

La integración continua ha llegado a los repositorios de Bitbucket, y todavía, de manera gratuita; aunque si bien es cierto, tiene sus limitaciones como el resto de servicios. Por ahora, nos permitirá la ejecución ininterrumpida de _scripts_ durante 2 horas donde contaremos con 4GB de RAM y 5GB de espacio en disco. La limitación mensual es de 500 minutos por usuario, y para los equipos de más de un usuario se multiplican estos minutos por cada uno, acumulando los minutos de ejecución.

La ejecución se hará sobre un contenedor _docker_, pudiendo seleccionar cuaquier imagen pública o privada de _Docker Hub_ o incluso nuestro propio contenedore hospedado con acceso público.

## ¿Cómo empezar?

Primero debemos habilitar **pipelines** en el repositorio y a continuación se nos mostrarán las diferentes plantillas a seleccionar dependiendo del lenguaje utilizado. El _script_ a ejecutar debe estar en el directorio raíz y llamarse **bitbucket-pipelines.yml**; este archivo indicará en formato _YAML_ la imagen de _docker_, la rama a la que afecta, los pasos a ejecutar...

https://confluence.atlassian.com/bitbucket/files/792298910/859457211/6/1490664866213/pipelines_yml_structure2.png


