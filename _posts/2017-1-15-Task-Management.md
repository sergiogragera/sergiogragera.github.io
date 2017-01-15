---
layout: post
title: IntelliJ IDEA + git-flow + Bitbucket issue tracker
published: true
---
## Sobre el plugin Task Management

[IntelliJ IDEA](https://www.jetbrains.com/idea) o cualquiera de sus IDEs permiten gestionar las tareas gracias a la incorporación del plugin **Task Management**. Este plugin permite conectar diferentes _trackers_, sin embargo, no existe la conexión con el _issue tracker de [Bitbucket](https://bitbucket.org)_ (aunque sí lo hace con Jira y Trello, siendo también productos de Atlassian).

Aquí explico cómo conectar este _tracker_ mediante la [API REST de Bitbucket](https://developer.atlassian.com/bitbucket/api/2/reference) y el plugin **Task Management** de IntelliJ IDEA.

## Integración del _issue tracker de Bitbucket_ en IntelliJ IDEA

Abrimos la gestión de _trackers_ desde el menú **Tools > Tasks & Contexts > Configure Servers...** Desde este menú observamos los diferentes servidores configurados. Podemos añadir cualquiera de los _trackers_ predefinidos o también el de tipo genérico (llamado **Generic**). En este ejemplo, utilizaremos el genérico para poder configurarlo con las llamadas REST de la API 2.0 de Bitbucket.

Una vez añadido el servidor, debemos configurar la conexión al mismo con la autenticación básica HTTP con usuario y contraseña (solo si tenemos la autenticación de verificación en dos pasos descativada, en caso contrario tendremos que autenticar con **OAuth2**):

  - **Server URL**: introducimos la url de autenticación que nos proporciona la API [https://api.bitbucket.org/2.0/user/repositories](https://api.bitbucket.org/2.0/user/repositories)
  - **Username**: nuestro usuario en Bitbucket.
  - **Password**: la contraseña de nuestro usuario en Bitbucket.
  - **Use HTTP authentication**: marcamos esta casilla para que realice la autenticación básica HTTP enviando usuario y contraseña a la URL especificada.
    
  <center><img src="{{ site.baseurl }}/images/general.png" alt="General" width="550"></center>

Tras esto, accedemos a la pestaña de **Server Configuration** donde debemos especificar las diferentes llamas REST para poder consultar las tareas en nuestro _tracker_.

  - **Login URL**: al haber definido autenticación HTTP este campo se encuentra deshabilitado y no necesitamos indicar ninguna URL.
  - **Tasks List URL**: en este caso indicaremos la llamada que debe hacer para resolver la lista de tareas, que como indica el API será una llamada GET a [https://api.bitbucket.org/2.0/repositories/{username}/{repo_slug}/issues](https://api.bitbucket.org/2.0/repositories/{username}/{repo_slug}/issues) (como veremos más adelante podemos utilizar variables de plantilla, que se declararán entre llaves).
  - **Single Task URL**: aquí indicaremos la llamada que se hará para resolver los datos de una tarea específica, utilizando la llamada GET a la API mediante la URL [https://api.bitbucket.org/2.0/repositories/{username}/{repo_slug}/issues/{id}](https://api.bitbucket.org/2.0/repositories/{username}/{repo_slug}/issues/{id})
  - Marcaremos el tipo de respuesta como **JSON**, que es el formato de la respuesta a las llamadas de la API.
  
  <center><img src="{{ site.baseurl }}/images/config.png" alt="Server Configuration" width="550"></center>

  - Marcaremos también la relación de cada campo esperado por el plugin **Tasks Management** con el campo que vendrá en la respuesta JSON. Las siguientes relaciones atañen a la respuesta de la llamada a **Tasks List URL** que contiene la lista de tareas; la primera es **tasks** e indicamos el nombre del elemento relacionado en la respuesta, que en en el caso del _tracker de Bitbucket_, viene en el elemento **values** (como se indica en el esquema especificado en la [API](https://developer.atlassian.com/bitbucket/api/2/reference/resource/repositories/%7Busername%7D/%7Brepo_slug%7D/issues)).
    
  - También podemos relacionar el **id** con el campo JSON **id** que marca el id de cada tarea en la respuesta; el campo **summary** lo relacionamos con **title** y el campo **description** con **description.raw**. Las siguientes relaciones con el prefijo singleTask indican lo mismo que las anteriores, pero atañen a la respuesta de la llamada a **Single Task URL**.
    
  <center><img src="{{ site.baseurl }}/images/mapping.png" alt="Mapping" width="550"></center>

  - Abrimos el gestor de variables de plantilla desde el botón **Manage Template Variables...** Aquí tenemos que indicar el valor de las variables usadas en las URLs anteriores y además indicaremos si aparece o no como campo editable para el usuario. Están definidas por defecto las dos básicas, como son **username** y **password**, pero además añadiremos **repo_slug** que indicará el nombre en código del repositorio y que lo utilizamos en las llamadas URL definidas anteriormente; y este campo lo haremos editable marcando la casilla de **Show on first tab** y le asignaremos como valor el nombre en código del repositorio.
    
  <center><img src="{{ site.baseurl }}/images/template_variables.png" alt="Template Variables" width="450"></center>

  Ahora, en la primera pestaña llamada **General** se nos mostrarán, además de los campos **username**, **password** ya comentados, el campo **repo_slug** que acabamos de insertar. 
  
  **NOTA:** podemos insertar también la variable de plantilla **state** y pasarla en la **Tasks List URL** como parámetro [https://api.bitbucket.org/2.0/repositories/{username}/{repo_slug}/issues?q=state=%22{state}%22](https://api.bitbucket.org/2.0/repositories/{username}/{repo_slug}/issues?q=state=%22{state}%22) La definimos en la primera pestaña **General** con el valor de **new** lo que hará que se filtren las tareas mostrando únicamente las que tienen este estado.

## Integración de _git-flow_ en IntelliJ IDEA

Para aquellos que usan la [filosofía _git-flow_](http://danielkummer.github.io/git-flow-cheatsheet) en el control de versiones, existe el plugin para IntelliJ IDEA que nos muestra y permite trabajar con _features_, _hotfix_ y _releases_ integrando todo esto con el plugin **Tasks Management**.

<center><img src="{{ site.baseurl }}/images/git_flow.png" alt="Git Flow" width="550"></center>

Como observamos en la siguiente imagen, al seleccionar un tarea pendiente (**Tools > Tasks & Contexts > + Open Task...**) se nos mostrará las opciones del control de versiones y también las opciones de _git-flow_. En este caso, deshabilitamos la opción de crear rama del VCS pero dejaremos habilitado la casilla de verificación que creará el _changelist de Git_ donde iran registrándose los cambios en el código. Además, activaremos ls opción Start feature o Start hotfix de _git-flow_ según sea nuestra necesidad en la tarea a realizar. Al aceptar, se creará la rama de _feature_ o _hotfix_ y la _chagelist de Git_.

<center><img src="{{ site.baseurl }}/images/open_task.png" alt="Open Task" width="550"></center>

Una vez finalicemos los cambios haremos el _commit_ del _changelist_, que únicamente contendrá los cambios relaizados para esta feature o hotfix. El mensaje de _commit_ que aparecerá será el definido en el plugin **Tasks Management**, en la pestaña **Commit Message**, que por defecto viene definido con la plantilla {id} {summary}.

Es importante saber que con este mensaje de _commit_ podemos ejecutar cambios de estados de las tareas automáticamente en el _issue tracker de Bitbucket_. Podemos usar las palabras _fix_, _resolve_, etc. al inicio del mensaje seguidas del id de la tarea (con el prefijo #) para que cuando se suba este _commit_ se resuelva la tarea de manera automática.

Por lo tanto, configuramos el **Commit Message** del plugin **Tasks Management** como la plantilla siguiente:

`fixed #{id} {summary}`

Al realizar el _commit_ del _changelist_, el mensaje por defecto será el de la plantilla anterior; tras este _commit_, podemos cerrar la _feature_ o _hotfix_, acabando con el comando `feature finish` o `hotfix finish` que de manera automática hará los merges entre las ramas de _master_ y _develop_.

## Flujo de trabajo

Una vez tenemos configurado el IDE con el gestor de tareas integrado y el plugin de _git-flow_ podemos seguir el flujo de trabajo siguiente:

- Desde el menú **Tools > Tasks & Contexts > + Open Task...** buscamos y abrimos la tarea a realizar.
- Seleccionamos crear la changelist y la opción feature o hotfix.
- Realizamos los cambios de código.
- Hacemos _commit_ (el mensaje está definido en la plantilla del Task Management y nos permite cambiar el estado de la tarea automáticamente cerrándola cuando sea publicada).
- Finalizamos la feature o hotfix desde el menú del plugin de IntelliJ IDEA.
- Realizamos el push de las diferentes ramas para acabar publicando los cambios en el repositorio.

<center><img src="{{ site.baseurl }}/images/master.png" alt="Git" width="550"></center>
