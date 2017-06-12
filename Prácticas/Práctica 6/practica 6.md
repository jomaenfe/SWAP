### Práctica 6: Discos en RAID.

**Configuración y automatizado de RAID:** En primer lugar, lo que debemos hacer antes de configurar un RAID en cualquier sistema es añadir discos duros (en el caso en el que sólo tengamos uno conectado.) En esta práctica en concreto se van a añadir dos discos más a parte del disco en el que se encuentra instalado el sistema operativo que mueve la máquina. Para emular el servidor yo he usado VMware y añadirlos es tan simple como editar los ajustes de la máquina virtual y añadir 2 discos más con un nombre distinto al que tenía el anterior. Una vez tenemos los discos añadidos arrancamos la máquina y, en principio, no deberíamos notar ningún cambio. Ahora debemos configurar los discos con respecto al software. Para ello vamos a usar la herramienta 'mdadm', para instalarla nos debería bastar con el comando `sudo apt-get install mdadm`. Una vez esté instalada deberemos seguir los siguientes pasos:
1. Obtener la información de los identificadores de los discos con la orden `sudo fdisk -l`. Obtendremos información como la que se muestra en la captura uno.
2. Una vez obtenemos la información que necesitamos y vemos que los discos han sido aceptados por el sistema empezamos a montar el RAID. Lo vamos a montar usando el dispositivo "/dev/md0". Para ello ejecutamos `sudo mdadm -C /dev/md0 --level=raid1 --raid-devices=2 /dev/sda /dev/sdb`. Con esta orden acabamos de montar el dispositivo en md0 pero en cuanto reiniciemos la máquina Linux lo renombrará automaticamente como md127.
3. Antes de reiniciar la máquina y habiendo ejecutado el comando anterior le daremos formato usando md0. Para ello ejecutamos `sudo mkfs /dev/md0` que por defecto lo inicializará con formato ext2.
4. Ahora creamos la carpeta en el sistema para que contenga la RAID. Primero creaos una carpeta `sudo mkdir /dat` y después montamos lo montamos con `sudo mount /dev/md0 /dat`.

Si hemos completado todos los pasos anteriores sin ningún problema ya deberíamos tener el RAID montado en nuestra máquina. Para comprobar que el raid está funcionando sin problemas ejecutamos `sudo mdadm --detail /dev/md0` y tendrá que sacarnos por pantalla información semejante a la que se ve en la captura dos. 

Si todo está funcionando bien podemos pasar a automatizar el montaje de la RAID al inicio del sistema para no tener que hacerlo nosotros manualmente. Esta configuración debe ser realizada sobre el archivo "fstab" que se encuentra en el directorio "/etc", lo único que tendremos que hacer es añadirle una línea con información relevante sobre la RAID y los dispositivos usados. Para ello vamos a seguir los siguientes pasos:

1. En primer lugar vamos a obtener la información sobre los dispositivos, para ello listamos el archivo "by-uuid" ejecutando `ls -l /dev/dsik/by-uuid`. Tras ejecutar esta orden observaremos por pantalla una lista similar a la que contiene la captura 3. 
2. Buscamos el identificador de md0 (o en su defecto, md127) y lo copiamos.
3. Ahora ejecutamos `sudo nano /etc/fstab` y añadimos la línea `UUID:<identificador del dispositivo> /dat ext2 defaults 0 0` donde hay que sustituir el identificador del dispositivo por el identificador real y "/dat" por la carpeta que hemos creado nosotros. 
4. Una vez hecho esto podemos reiniciar la máquina tranquilamente que se montará automaticamente el RAID en el arranque. Para comprobar que se esta ejecutando al arracnar el sistema podemos usar la orden que hemos usado en la captura dos.


![Captura 1](http://imgur.com/Yq4sxkH.jpg "Captura 1")
Captura 1.

![Captura 2](http://imgur.com/afMNGpO.jpg "Captura 2")
Captura 2.

![Captura 3](http://imgur.com/E6TMG2F.jpg "Captura 3")
Captura 3.

**Simular fallos y comprobar que todo funciona correctamente:** Para probar el funcionamiento del RAID antes de ponerlo a funcionar de fomra seria podemos ocasionar fallos por software y comprobar como reacciona. En este apartado vamos a probar diferentes tipos de fallos. Primero vamos a ocasionar un fallo en el disco `sudo mdadm --manage --set-faulty /dev/md127 /dev/sdb`, debería mostrar por pantalla unas líneas como las que se ven en la captura cuatro.

![Captura 4](http://imgur.com/AbaLkLg.jpg "Captura 4")
Captura 4.

Para provocar el siguiente fallo podemos retirar un disco en "caliente" (es decir, con la máquina arrancada y funcinando), para ello ejecutaremos `sudo mdadm --manage --remove /dev/md127 /dev/sdb`. Para comprobar que se ha extraido correctamente comprobarmos las líneas que nos muestra el terminal con las que hay en la captura cinco.


![Captura 5](http://imgur.com/6Qs4xxQ.jpg "Captura 5")
Captura 5.

Y por último vamos a volver a conectar el disco en caliente, para ello ejecutamos `sudo mdadm --manage --add /dev/md0 /dev/sdb` y volvemos a comprobar con la captura seis las líneas de terminal.

![Captura 6](http://imgur.com/gV8XyrS.jpg "Captura 6")
Captura 6.

Cada vez que produzcamos un error es recomendable que comprobemos el estado del raid con la orden que usamos en la captura dos para asegurarnos de que todo funciona correctamente. 
