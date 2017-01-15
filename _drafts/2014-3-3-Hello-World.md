---
layout: post
title: IntelliJ IDEA + git-flow + Bitbucket issue tracker
published: true
---
IntelliJ IDEA o cualquiera de sus IDEs permiten gestionar las tareas gracias a la incorporación del plugin **Task Management**. Este plugin permite conectar diferentes _trackers_, sin embargo, no existe la conexión con el _issue tracker de Bitbucket_ (aunque sí lo hace con Jira y Trello, ambos productos también de Atlassian).

Aquí explico cómo conectar este _tracker_ mediante la API REST de Bitbucket y el plugin **Task Management** de IntelliJ IDEA.

# Integración del _issue tracker de Bitbucket_ en IntelliJ IDEA

Abrimos la gestión de _trackers_ desde el menú **Tools > Tasks & Contexts > Configure Servers...** Desde este menú observamos los diferentes _trackers_ configurados. Podemos añadir cualquiera de los predefinidos o también el genérico, llamado **Generic**. En este ejemplo, utilizaremos el genérico para poder configurarlo con las llamadas REST de la API 2.0 de Bitbucket.

##Referencia a la API REST 2.0 de Bitbucket

1. Una vez añadido el servidor, debemos configurar la conexión al mismo con la autenticación básica HTTP con usuario y contraseña.

	1. **Server URL**: aquí introducimos la url de autenticación que nos proporciona la API https://api.bitbucket.org/1.0/user/repositories

	2. **Username**: nuestro usuario en Bitbucket.

	3. **Password**: la contraseña de nuestro usuario en Bitbucket.

	4. **Use HTTP authentication**: marcamos esta casilla para que realice la autenticación básica HTTP enviando usuario y contraseña a la URL especificada.
    
    ##Captura de la pestaña General

2. Tras esto, accedemos a la pestaña de **Server Configuration** donde debemos especificar las diferentes llamas REST para poder consultar las _issues_ en nuestro _tracker_.

	1. **Login URL**: al haber definido autenticación HTTP este campo se encuentra deshabilitado y no necesitamos indicar ninguna URL.

	2. **Tasks List URL**: en este caso indicaremos la llamada que debe hacer para resolver la lista de tareas, que como indica el API será una llamada GET a https://api.bitbucket.org/1.0/repositories/{username}/{repo_slug}/issues (como veremos más adelante podemos utilizar variables de plantilla, que se declararán entre llaves).

	3. **Single Task URL**: aquí indicaremos la llamada que se hará para resolver los datos de una tarea específica, utilizando la llamada GET a la API mediante la URL https://api.bitbucket.org/1.0/repositories/{username}/{repo_slug}/issues/{id}

	4. Marcaremos el tipo de respuesta como **JSON**, que es el formato de la respuesta a las llamadas de la API.

	##Captura de la pestaña Server Configuration

	5. Marcaremos también la relación de cada campo esperado por el plugin **Tasks Management** con el campo que vendrá en la respuesta JSON. Las siguientes relaciones atañen a la respuesta de la llamada a **Tasks List URL** que contiene la lista de tareas; la primera es **tasks** e indicamos el nombre del elemento relacionado en la respuesta, que en en el caso del _tracker de Bitbucket_, viene en el elemento **issues**, como se puede observar en la siguiente imagen.
    
    ##Captura de pantalla del JSON de respuesta
    
    También podemos relacionar el **id** con el campo JSON **local_id** que marca el id de cada tarea en la respuesta; el campo **summary** lo relacionamos con **title** y el campo **description** con **content**. Las siguientes relaciones con el prefijo singleTask indican lo mismo que las anteriores, pero atañen a la respuesta de la llamada a **Single Task URL**.
    
    ##Captura de tabla de mappings

	6. Abrimos el gestor de variables de plantilla desde el botón **Manage Template Variables...** Aquí tenemos que indicar el valor de las variables usadas en las URLs anteriores y además indicaremos si aparece o no como campo personalizable para el usuario. Están definidas por defecto las dos básicas, como son **username** y **password**, pero además nosotros añadiremos **repo_slug** que indicará el nombre en código del repositorio; y este campo lo haremos editable marcando la casilla de **Show on first tab**.
    
    ##Captura de la tabla de template variables

Ahora, en la primera pestaña llamada **General** se nos mostrarán, además de los campos **username**, **passwrod** ya comentados, el campo **repo_slug** que acabamos de insertar. **NOTA:** podemos insertar también la variable de plantilla status en la URL y definirla en la primera pestaña **General** con el valor de **new** lo que hará que se filtren las tareas mostrando únicamente las que tienen este estado.

# Integración de _git-flow_ en IntelliJ IDEA

Para aquellos que usan la filosofía _git-flow_ (http://danielkummer.github.io/git-flow-cheatsheet) en el control de versiones, existe el plugin para IntelliJ IDEA que nos muestra y permite trabajar con _features_, _hotfix_ y _releases_ integrando todo esto con el plugin **Tasks Management**.

##Captura del plugin _git-flow_

Como observamos en la siguiente imagen, al seleccionar un tarea pendiente se nos mostrará las opciones del control de versiones y también las opciones de _git-flow_. En este caso, deshabilitamos la opción de crear rama del VCS pero dejaremos habilitado la casilla de verificación que creará el _chagelist de Git_ donde iran registrándose los cambios en el código.

Además, activaremos ls opción Start feature o Start hotfix de _git-flow_ según sea nuestra necesidad en la tarea a realizar. Al aceptar, se creará la rama de _feature_ o _hotfix_ y la _chagelist de Git_.

Una vez finalicemos los cambios haremos el commit del chagelist, que únicamente contendrá los cambios relaizados para esta feature o hotfix. El mensaje de commit que aparecerá será el definido en el plugin **Tasks Management**, en la pestaña **Commit Message**, que por defecto viene definido con la plantilla {id} {summary}.

# Cambio de estado en Bitbucket

Es importante saber que con este mensaje de commit podemos ejecutar cambios de estados de las tareas automáticamente. Al usar el issue tracker de Bitbucket, podemos usar las palabras fix, resolve, etc. al inicio del mensaje seguidas del id de la tarea (con el prefijo #) para que cuando se suba este commit en el repositorio, se resuelva la tarea con este id automáticamente.

##Captura o referencia a las palabras usadas

Por lo tanto, configuramos el **Commit Message** del plugin **Tasks Management** como la plantilla siguiente:

fixed #{id} {summary}

Al realizar el commit del chagelist, el mensake por defecto será el de la plantilla anterior; tras este commit, podemos cerrar la feature o hotfix, acabando con el comando feature finish o hotfix finish que de manera automática hará los merges entre las ramas de master y develop.

# Flujo de trabajo

Una vez tenemos configurado el IDE con el gestor de tareas integrado y el plugin de _git-flow_ podemos seguir el flujo de trabajo siguiente:

1. Desde el menú **Tools > Tasks & Contexts > Open Task...** buscamos y abrimos la tarea a realizar.
2. Seleccionamos crear la changelist y la opción feature o hotfix.
3. Realizamos los cambios de código.
4. Hacemos commit (el mensaje está definido en la plantilla del Task Management y nos permite cambiar el estado de la tarea automáticamente cerrándola cuando sea publicada).
5. Finalizamos la feature o hotfix desde el menú del plugin de IntelliJ IDEA.
6. Realizamos el push de las diferentes ramas para acabar publicando los cambios en el repositorio.

##Captura de pantalla de git con la subida de una feature

![_config.yml]({{ site.baseurl }}/images/config.png)

The easiest way to make your first post is to edit this one. Go into /_posts/ and update the Hello World markdown file. For more instructions head over to the [Jekyll Now repository](https://github.com/barryclark/jekyll-now) on GitHub.
