### Práctica 2: Clonar la información de un sitio web.

**Copia de archivos por ssh:** Para realizar copias de archivos con ssh, podemos usar la orden "scp". Más concretamente la línea `scp hola.html usuario@maquinadestino:/home/usuario` esta orden copiaría el archivo "hola.html" desde la máquina en la que se está enviando hasta la máquina de destino a la que se está enviando. Podemos ver el ejemplo en la captura 1, al principio en la máquina 2 sólo estaba el archivo index.html y después de ejecutar la orden "scp" en la máquina 1 se transfiere el archivo.

![Captura 1](http://i.imgur.com/hTTPzgy.jpg "Captura 1")
Captura 1

**Herramienta rsync:** En primer lugar lo que tenemos que hacer es probar la herramienta rsync para clonar la información que habrá en la maquina principal a la máquina secundaria. Para ello usamos el comando `rsync -avz -e ssh root@maquina1:/var/www/ /var/www/` donde tenemos que sustituir maquina1 por la dirección Ip de la máquina1 y con esto conseguimos que se copie la carpeta /var/www y todo su contenido de una máquina a otra. 

![Captura 2](http://i.imgur.com/wr83Xv4.jpg "Captura 2")
Captura 2

Para hacer una prueba he creado un archivo "hola.html" en la maquina principal, en la ruta donde apache2 es capaz de mostrarlo. Como se ve en la Captura 2, en la máquina principal se encuentra dicho archivo y en la secundaria no.

![Captura 3](http://i.imgur.com/Bl87ALt.jpg "Captura 3")
Captura 3

Al ejecutar el comando que muestro arriba, en la captura 3, la maquina secundaria solicita la información de ese directorio a la primera. La primera se lo envía y la máquina secundaria lo clona exactamente igual, es decir, con todos los permisos y demás información sobre el archivo.

**Acceso sin contaseña para ssh:** Como vamos a crear una tarea en la que rsync se encargará de sincronizar automáticamente el contenido no podemos dejar que la tarea se quede parada esperando la contraseña para poder conectarse a la máquina princiapl y transferir los cambios que se hayan realizado. Para ello generamos una clave de tipo rsa con la orden `ssh-keygen -b 4096 -t rsa`. Una vez se ha generado este tipo de clave la transferimos a la máquina 1 con la orden `ssh-copy-id maquina1` donde sustituimos maquina1 por la dirección IP de dicha máquina. Una vez hecho esto y si la transferencia ha sido exitosa nos dejará acceder sin contraseña mediante ssh a la máquina principal como se puede observar en la captura .

![Captura 4](http://i.imgur.com/6mWQcf0.jpg "Captura 4")
Captura 4

**Programando tareas en crontab:** Crontab es una utilidad del sistema que sirve para programar tareas que se ejecutarán en segundo plano en un día y hora concretas. Para visualizarlo debemos teclear la orden `cat /etc/crontab`, con esto nos mostrará en terminal una imagen como la que se aprecia en la captura 5. Para poder editarlo debemos usar la orden `sudo nano /etc/crontab`.

![Captura 5](http://i.imgur.com/npLfkb1.jpg "Captura 5")
Captura 5

Como se ve en la captura 5, he creado una tarea que a cada hora hace un "backup" de la máquina principal en la máquina secundaria. La sintaxis de la línea significa que en el minuto 1 de cada, hora, día y més del año se ejecuta como root la orden "rsync" para hacer la sincronización.