---
published: false
---
## SSH _tunneling_

Entre las opciones a la hora de conectar mediante SSH (Secure SHell) está la pposibilidad de crear un túnel desde la máquina local hasta una tercera máquina.

Supongamos que tenemos acceso SSH a una máquina donde tenemos nuestro servidor; sin embargo, no tenemos acceso directo a la máquina donde tenemos la base de datos. Esta máquina es solo accesible desde la máquina donde está nuestro servidor, pero no desde nuestra máquina local. Por tanto, podemos conectar a la máquina donde está el servidor y desde aquí crear una conexión a base de datos remota ya que sí que tenemos acceso a está tercera máquina.

### Esquema explicando A -> B -> C pero no A -> C

Sin embargo, en el flujo de trabajo del desarrollo de software parece interesante tener las conexiones a las bases de datos abiertas desde el propio IDE, como ya se integra desde IntelliJ IDEA. Por tanto, para poder conectar desde la máquina local (la máquina de desarrollo) necesitamos acceso a través de la máquina donde se aloja el servidor, creando un túnel hasta la máquina remota.

`ssh user@maquina.servidor -L 2000:maquina.base.datos:1521`

¿Qué significa este último argumento? Significa que todo lo que enviemos por nuestro puerto puerto de salida 2000 irá a la máquina servidor y saltará a la máquina con la base de datos entrando por está última por el puerto 1521. Con lo cual, si en la tercera máquina tenemos un Oracle escuchando en el puerto 1521, desde nuestra máquina local de desarrollo podemos crear una conexión a la base de datos remota conectando a localhost:2000 (cuyas peticiones, como acabamos de explicar, llegarán a la tercera máquina por el puerto 1521 donde está escuchando Oracle).

### Usando el conector de base de datos de IntelliJ IDEA con un túnel SSH

Como hemos dicho, desde el IDE podemos crear una conexión a la base de datos creando la conexión a local por el puerto 2000 cuando tenemos el túnel SSH abierto. Pero también podemos configurar la conexión abriendo el túnel SSH desde la propia configuración de la conexión a BBDD.

### Captura de conexión a BBDD mediante IntelliJ IDEA

Abrimos **path -> a conectores** donde tenemos las conexiones a base de datos. Creamos entonces una nueva conexión seleccionando el driver según la necesidad; en este ejemplo usaremos el driver de Oracle. Accedemos ahora a la pestaña **config** y debemos introducir los datos del túnel SSH a crear cuando abramos esta conexión.

### Captura SSH túnel config

Por último, desde la pestaña **primera** introduciremos los datos de la conexión a la tercera máquina, la de la base de datos, ya que tendremos acceso puesto que el IDE abrirá un túnel mediante la conexión a la máquina del servidor que hará de _proxy_. No especificamos aquí el puerto local por donde salir en el túnel, pero IntelliJ IDEA está haciendo esto transparente para nosotros.

## SSH _reverse tunneling_

Explicar el caso de Single SigOn, donde no podemos definir la url de redirección como localhost:9000 pero podemos crear en una máquina accesible publicamente un túnel reverso SSH a un puerto local donde tenemos el servidor depurando la aplicación que desarrollamos.

`ssh -R 8000:localhost:9000 user@maquina.accesible.publica`

Ahora, todo lo que salga de la máquina remota por el puerto 8000, llegará a nuestro puerto local 9000 donde está nuestro servidor de desarrollo. Por tanto, podemos configurar la URL de redirección del servicio www.single.sign.on a la máquina accesible públicamente con el puerto 80, por ejemplo.

Entonces, finalmente, cuando lleguen peticiones al puerto 80, esas llamadas tienen que ser redirigidas hacia el puerto 8000, haciendo así que salgan hacia nuestra máquina local, llegando por el puerto 9000 donde serán atendidas por nuestro servidor. Aquí debemos crear un túnel por el que todo lo que venga por el puerto 80 debe ser redirigido al puerto 8000 que conectará con el puerto 9000 de la máquina de depuración.

`ssh -L maquina.accesible.publica:7000:localhost:8000 user@maquina.accesible.publica`

_Ver si hace falta hacer el VirtualHost, y en caso de ser necesario no se necesitará este nuevo túnel_

## localtunnel

Si en el caso anterior no disponemos de una máquina accesible publicamente, podemos instalar el comando [localtunnel](https://localtunnel.github.io/www/) y así efectuar las acciones explicadas en el apartado anterior pero conectando con una máquina ajena a nosotros cuyo subdominio será creado dinamicamente cada vez que levantemos el servicio.

Así solo tendremos que ejecutar `lt --port 9000 --subdomain test` y esto nos creará una conexión pública a nuestra máquina local en el puerto 9000. La URL de conexión generada será test.localtunnel.me, y accediendo a [https://test.localtunnel.me](https://test.localtunnel.me) llegaremos a nuestra máquina por el puerto 9000, donde nos atenderá nuestro servidor de aplciaciones.
