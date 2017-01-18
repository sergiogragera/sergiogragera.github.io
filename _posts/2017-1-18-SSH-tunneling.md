---
layout: post
title: SSH tunneling
published: true
---
Entre las opciones a la hora de conectar mediante SSH (Secure SHell) existe la pposibilidad de crear un túnel desde la máquina local hasta una tercera máquina.

Supongamos que tenemos acceso SSH a una máquina donde tenemos nuestro servidor; sin embargo, no tenemos acceso directo a la máquina donde tenemos la base de datos. Esta máquina es solo accesible desde la máquina donde está nuestro servidor, pero no desde nuestra máquina local actual en la que estamos trabajando.

### Esquema explicando A -> B -> C pero no A -> C

Por tanto, para poder conectar desde la máquina local necesitamos acceso a través de la máquina donde se aloja el servidor, creando un túnel hasta la máquina remota.

`ssh usuario@maquina.servidor -L 2000:maquina.base.datos:1521`

¿Qué significa este último argumento? Significa que todo lo que enviemos por nuestro puerto puerto de salida 2000 irá a la máquina servidor y saltará a la máquina con la base de datos entrando en esta última por el puerto 1521. Con lo cual, si en la tercera máquina tenemos un Oracle escuchando en el puerto 1521, desde nuestra máquina local podemos crear una conexión a la base de datos remota conectando a localhost:2000 (cuyas peticiones, como acabamos de explicar, llegarán a la tercera máquina, a través del túnel SSH, por el puerto 1521 donde está escuchando Oracle).

<center><img src="{{ site.baseurl }}/images/ssh_1.png" alt="Conexión a la base de datos con SSH tunneling" width="650"></center>

Como se muestra en la imagen anterior, desde el IDE podemos crear una conexión a la base de datos creando la conexión a local por el puerto 2000 cuando tenemos el túnel SSH abierto. Pero también podemos configurar la conexión abriendo el túnel SSH desde la propia configuración de la conexión a BBDD en IntelliJ IDEA.

Abrimos **View -> Tool Windows -> Database** que nos mostrará la ventana de administración de las conexiones a base de datos. Creamos entonces una nueva conexión seleccionando añadir un nuevo _data source_, en este ejemplo usaremos el de Oracle. Accedemos ahora a la pestaña **SSH/SSL** y debemos introducir los datos del túnel SSH a crear cuando abramos esta conexión.

<center><img src="{{ site.baseurl }}/images/ssh_2.png" alt="Configuración de la conexión con SSH tunneling" width="650"></center>

Por último, desde la pestaña **General** introduciremos los datos de la conexión a la tercera máquina, la de la base de datos, ya que tendremos acceso puesto que el IDE abrirá un túnel mediante la conexión a la máquina del servidor que hará de _proxy_. No especificamos aquí el puerto local por donde salir en el túnel, pero IntelliJ IDEA está haciendo esto transparente para nosotros.

<center><img src="{{ site.baseurl }}/images/ssh_3.png" alt="Conexión a base de datos" width="650"></center>

## SSH _reverse tunneling_

Por otra parte, tenemos un servicio de Single Sign-On ajeno a nosotros donde no podemos definir la url de redirección como localhost:9000, pero podemos crear en una máquina accesible publicamente un túnel reverso SSH a un puerto local (la máquina que estamos desarrollando, es decir, localhost:9000 por ejemplo) donde tenemos el servidor depurando la aplicación que desarrollamos.

`ssh -R maquina.accesible.publica:8000:localhost:9000 usuario@maquina.accesible.publica`

Ahora, todo lo que salga de la máquina remota por el puerto 8000, llegará a nuestro puerto local 9000 donde está nuestro servidor de desarrollo. Por tanto, podemos configurar la URL de redirección del servicio www.single.sign-on a la máquina accesible públicamente por el puerto 80.

Esas llamadas que llegan al puerto 80 tienen que ser redirigidas hacia el puerto 8000, haciendo así que salgan hacia nuestra máquina local, llegando por el puerto 9000 donde serán atendidas por nuestro servidor. 

```
<VirtualHost *:80>
    ServerName maquina.accesible.publica

    ProxyPreserveHost On
    ProxyRequests Off

    <Proxy *>
        Order deny,allow
        Allow from all
    </Proxy>

    ProxyPass / ajp://maquina.accesible.publica:8000/
    ProxyPassReverse / ajp://maquina.accesible.publica:8000/
    ServerAlias maquina.accesible.publica
</VirtualHost>
```

<center><img src="{{ site.baseurl }}/images/ssh_4.jpg" alt="Caos SSH tunneling" width="450"></center>

## localtunnel

Si en el caso anterior no disponemos de una máquina accesible publicamente, podemos instalar el comando [localtunnel](https://localtunnel.github.io/www/) y así efectuar las acciones explicadas en el apartado anterior pero conectando con una máquina ajena a nosotros cuyo subdominio será creado dinamicamente cada vez que levantemos el servicio.

Así solo tendremos que ejecutar `lt --port 9000 --subdomain test` y esto nos creará una conexión pública a nuestra máquina local en el puerto 9000. La URL de conexión generada será test.localtunnel.me, y accediendo a [https://test.localtunnel.me](https://test.localtunnel.me) llegaremos a nuestra máquina por el puerto 9000, donde nos atenderá nuestro servidor de aplciaciones.
