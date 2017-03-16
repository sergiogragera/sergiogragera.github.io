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

###¿Cómo los ejecutamos?

Cuando en Maven ejecutamos `mvn clean` estamos ejecutando el plugin _maven-clean-plugin_ que incorpora nativamente la herramienta. Este plugin tiene un único _goal_ que son las acciones posibles a ejecutar, y en este caso ese _goal_ es 'clean' que tiene por objetivo eliminar el directorio _target_ de nuestros módulos. Además del _goal_ los _plugins_ tienen una eqtiqueta _executions_ que permite definir las diferentes ejecuciones del _plugin_, por ejemplo indicando diferentes fases donde ejecutarlo. Por otra parte, el _plugin_ puede configurarse con variables parametrizadas asignándose los valores en cada ejecución o de manera global.

```xml
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-clean-plugin</artifactId>
            <version>2.5</version>
            <executions>
                <execution>
                    <phase>clean</phase>
                    <goals>
                        <goal>clean</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
```

Para ejecutar un plugin solo tenemos que invocarlo desde el comando `mvn` indicando el _plugin_ y _goal_ a ejecutar. Siguiendo con el ejemplo anterior podemos ejecutar el siguiente comando:

```bash
        mvn org.apache.maven.plugins:maven-clean-plugin:2.5:clean
```

Al ser un plugin propio de Maven (sigue la convención de nomenclatura oficial **maven-_name_-plugin** y pertenece al paquete _org.apache.maven.plugins_) podemos invocarlo solo indicando el nombre (_name_ indicado entre las palabras maven y plugin), quedando la ejecución anterior como:

```bash
        mvn clean:clean
```

En Gradle ocurre algo similar. Tenemos por una parte la declaración del _plugin_ y la declaración de la aplicación o ejecución del mismo. En el caso siguiente tenemos la declaración de un plugin que no incluye Gradle por defecto como es el desarrollado por [SourceClear](https://app.sourceclear.com) y por tanto primero se define como dependencia y luego se aplica; por otro lado está la aplicación del _plugin_ **idea** que incluye la herramienta.

```groovy
        plugins {
            id 'com.srcclr.gradle' version '2.0.1'
        }
        
        apply plugin: 'com.srcclr.gradle'
        apply plugin: 'idea'
```

Cada _plugin_ define una tareas (lo que homologamente en Maven son los _goals_) y por tanto, la aplicación de un plugin implica la disposición de ejecutar estas tareas. Si queremos ejecutarlas en una fase concreta, o hacerla dependiente de la ejecución de otra tarea la podremos definir en el archivo _build.gradle_ como sigue:

```groovy
        clean.dependsOn(cleanIdea)
        
        clean.finalizedBy(cleanIdea)
```

Y podemos ejecutar cualquier tarea ejecutando el comando _gradle_ seguido del nombre de la tarea:

```groovy
        gradle idea
```

###¿Cuándo se ejecutan?

En Maven, los _plugins_ se ejecutan en las diferentes fases de construcción del proyecto agrupadas en tres ciclos de vida diferentes: _clean_, _default_ y _site_. Sin entrar en más detalle (véase [Maven Lifecycle Reference](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html#Lifecycle_Reference)) tenemos fases dentro del ciclo de vida _default_ como son _process-resources_ (momento en el cual se copian los recursos en el directorio _target_), _compile_, _test_, _package_ (empaquetado del código compilado en formato JAR, por ejemplo) o _verify_ (comprobación de que los distintos controles como test unitarios, de integración o cuaquier otra comprobación se completa correctamente cuando se procesa el empaquetado cumpliendo así los criterios de calidad). La etiqueta que nos permite definir la fase de ejcución es _phase_ y estará definida dentro de la etiqueta de ejecución, en las múltiples ejecuciones posibles a definir.

```xml
        <executions>
            <execution>
                <phase>process-resources</phase>
                <goals>
                    <goal>compress</goal>
                </goals>
            </execution>
        </executions>
```

En Gradle solo hay tres fases, de inicialización, de configuración y de ejecución. Sin embargo aquí se introducen las tareas o _Tasks_ las cuales ejecutan acciones. Estas tareas pueden ser ejecutadas según el orden deseado e introduciendo dependencias de orden entre ellas. Los plugins introducen nuevas tareas que son ejecutadas en el momento de ejecución y nuestras acciones pueden asociarse a la ejecución posterior o anterior a estas tareas.

```groovy
        compileJava.finalizedBy(combineJs, minifyJs)
```

###¿Cómo los configuramos?

Los plugins aceptan parámetros y por tanto estos deben ser definidos, configurando así la ejecución del _plugin_. En el caso de Maven, tenemos la etiqueta _configuration_ que nos permite declarar los valores con los que queremos inicializar los parámetros que expone el _plugin_. Si lo hacemos a nivel de la etiqueta _execution_ estos valores solo se inicializarán cuando se ejecute esta ejecución, sin embargo también se nos permite declarar esta configuración en el nivel anterior, lo que hará que se utilicen por defecto estos valores aquí definidos.

```xml
        <configuration>
            <inputDir>${basedir}/src/main/webapp/</inputDir>
            <output>${build.outputDirectory}</output>
        </configuration>
```

En Gradle tenemos también una manera de configurar el _plugin_ y por tanto definir los valores de los parámetros que expone dicho _plugin_.

```groovy
        idea {
            module {
                iml {
                    generateTo = file('secret-modules-folder')
                }
            }
        }
```

##Desarrollando _plugins_ con Java

###Maven

Lo primero que tenemos que definir en el _pom.xml_ de nuestro proyecto _plugin_ (el cual deberá llamarse _name_-mave-plugin para cumplir con la nomenclatura propuesta por Maven y diferenciar de los _plugins_ oficiales de Maven que serán maven-_name_-plugin) es el tipo de empaquetado que hará Maven que será de tipo **maven-plugin**:

```xml
        <groupId>groupId</groupId>
        <artifactId>myname-maven-plugin</artifactId>
        <version>1.0.0</version>
        <packaging>maven-plugin</packaging>
```

Por otra parte es necesario importar las siguientes librerías de Maven:

```xml
        <dependencies>
            <dependency>
                <groupId>org.apache.maven</groupId>
                <artifactId>maven-plugin-api</artifactId>
                <version>3.0</version>
            </dependency>

            <dependency>
                <groupId>org.apache.maven.plugin-tools</groupId>
                <artifactId>maven-plugin-annotations</artifactId>
                <version>3.4</version>
                <scope>provided</scope>
            </dependency>
        </dependencies>
```

Finalmente cumpliremos con la API de los plugins de Maven implementando una clase Mojo (Maven plain Old Java Object) la cual extenderá de _AbstractMojo_ e implementará el método _execute_. Para definir la clase como Mojo deberemos indicar la anotación sobre el nombre de la clase, indicando el nombre del Mojo (esto será el nombre del _goal_) y la fase por defecto en la que se ejecutará (si no indicamos ninguna fase el plugin no se ejecutará cuando por ejemplo se ejecute un _build_ y solo se ejecutará si lo lanzamos manualmente).

```java
        @Mojo(name = "goalName", defaultPhase = LifecyclePhase.COMPILE)
        public class MyMojo extends AbstractMojo {
            @Parameter(property = "directory", defaultValue = "src/main/webapp")
            private String directory;

            @Override
            public void execute() throws MojoExecutionException, MojoFailureException {
                //Llamada a la clase que ejecuta las acciones de nuestro plugin, 
                //donde podemos pasarle el parámetro de configuración directory
                //que puede venir con un valor o por defecto ser src/main/webapp
            }
        }
```

Una vez ejecutemos `mvn install` ya tendremos disponible el _plugin_ en el repositorio local de Maven. Por lo que podremos ejecutar el siguiente comando (podemos obviar la versión ya que Maven utilizará la última existente):

```
        mvn groupId:myname-maven-plugin:1.0.0:goalName
```


###Gradle

Para crear un _plugin_ de Gradle debemos definir en el archivo de configuración _build.gradle_ la dependencia a la api de Gradle:

```groovy
        dependencies {
            compile gradleApi()
        }
```

Una vez definida la dependencia ya podremos crear la clase _plugin_ que será la que defina las diferentes tareas que expone nuestro _plugin_:

```java
        import org.gradle.api.Plugin;
        import org.gradle.api.Project;
        
        public class MyPlugin implements Plugin<Project> {
            public void apply(Project project) {
                project.getExtensions().create("myplugin", MyExtension.class);
                project.getTasks().create("myplugin", MyTask.class);
            }
        }
```

En la clase anterior estamos diciendo que nuestro plugin tendrá una tarea llamada 'myplugin' y una extensión (no es más que una configuración) que se definirá bajo la etiqueta 'myplugin' (por convencionalismos llamaremos a la configuración de la tarea con el mismo nombre que la tarea).

Necesitamos entonces definir la clase extensión que recibirá los valores iniciales de los parámetros configurables:

```java
        public class MyExtension {
            private String directory = "src/main/webapp";

            public String getDirectory() {
                return directory;
            }

            public void setDirectory(String directory) {
                this.directory = directory;
            }
        }
```

Finalmente debemos definir nuestra tarea en la clase MyTask que hará uso de la configuración leída por la extensión anterior.

```java
        import org.gradle.api.DefaultTask;
        import org.gradle.api.tasks.TaskAction;

        public class JugCsTask extends DefaultTask {
            @TaskAction
            public void myTaskAction() {
                MyExtension extension = getProject().getExtensions().findByType/MyExtension.class);
                if (extension == null) {
                    extension = new MyExtension();
                }

                String directory = extension.getDirectory();

                //Llamada a la clase que ejecuta las acciones de nuestro plugin, 
                //donde podemos pasarle el parámetro de configuración directory
                //que puede venir con un valor o por defecto ser src/main/webapp
            }
        }
```

Una vez tenemos todas las clases anteriores definidas solo nos queda indicar a gradle que esto es un _plugin_ y la clase que define al _plugin_; concretamente la clase que define las tareas y extensiones de nuestro _plugin_, que en este caso es MyPlugin.java. Esto lo haremos creando un directorio **gradle-plugins** bajo el directorio src/main/resources/META-INF donde deberemos especificar en el archivo de _properties_ (el nombre de este archivo debe ser el nombre completo con el que se llamará al _plugin_) la clase inicial donde se definen las tareas:

```
        src/
          main/
            resoruces/
              META-INF/
                gradle.plugins/
                  groupId.myplugin.properties
```

Este archivo contendrá la siguiente línea:

```
        implementation-class=MyPlugin
```

Realizando la instalación en el repositorio local de MAven ejecutando el comando _install_ del _plugin_ de _maven_ tendremos la posibilidad de importarlo, aplicarlo y usarlo desde otros proyectos. Para esto, en el _build.gradle_ debemos tener el _plugin_ de Maven y Java:

```
        group 'groupId'
        version '1.0.0'

        apply plugin: 'java'
        apply plugin: 'maven'

        sourceCompatibility = 1.8

        repositories {
            mavenCentral()
        }

        dependencies {
            compile gradleApi()
        }
```

Y desde cualquier proyecto podremos importar este _plugin_ de la siguiente manera:

```
        buildscript {
            repositories {
                mavenLocal()
            }
            dependencies {
                classpath("groupId:myplugin:1.0.0")
            }
        }
        
        apply plugin: 'groupId.myplugin' //Nombre del archivo bajo el directorio META-INF/gradle-plugins
```
