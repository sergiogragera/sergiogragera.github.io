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

##Desarrollando _plugins_ con Java

###Maven



###Gradle
