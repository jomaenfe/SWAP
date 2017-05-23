### Practica 4: Asegurar la granja web

**Instalación de un certificado SSL autofirmado:** Un certificado sirve para dar seguridad a los usuarios de una página web de que esa misma página web es segura, auténtica y está administrada por la empresa que la está ofreciendo. Nosotros vamos a crear un certificado usando el protocolo SSL, un protocolo de comunicación que se ubica en la pila de protocolos sobre TCP/IP. Este protocolo proporciona servicios de comunicación segura entre cliente y servidor, entre ellos autenticación, integridad y privacidad. Para proceder a la instalación primero debemos tener instalado un servidor web sobre el que certificar la información, en mi caso tengo instalado apache en ubuntu server. Para certificar información primero activamos el módulo de apache con la orden `a2enmod ssl` una vez hecho esto reinicamos el servicio de apache `service apache2 restart`. Ahora creamos una carpeta en la carpeta donde está instalado apache `mkdir /etc/apache2/ssl` y dentro de esta carpeta creamos el archivo "apache.key" y "apache.crt" con la orden `openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/apahce2/ssl/apache.key -out /etc/apache2/ssl/apache.crt`. Después de ejecutar esta orden nos pedirá un conjunto de datos sobre nuestra información que debemos rellenar. Por último, editamos el archivo de configuración de apache `sudo nano /etc/apache2/sites-availiable/default-ssl` y agregamos las siguientes lineas debajo de done pone "SSLEngine on", `SSLCertificateFile /etc/apache2/ssl/apache.crt`
`SSLCertificateKeyFile /etc/apache2/ssl/apache.key` una vez hecho esto ejecutamos `a2ensite default-ssl` para recargar la configuración y `service apache2 reload` para recargar el servicio.
Como se puede ver en la captura al acceder a la página web del servidor nos dice el certificado no es fiable por lo que está funcionando.

![Captura 1](http://imgur.com/7ZN46iS.jpg "Captura 1")
Captura 1.

**Configurar las normas de acceso al servidor mediante Iptables:** En esta parte de la práctica vamos a crear un script que se ejecutará al arrancar el sistema, este script va a impedir la entrada y salida de tráfico de red por cualquier puerto que no sea el del servidor. En el script lo que se ha añadido son reglas que primeramente limpian configuración que pueden haber dejado reglas anteriores ejecutadas en el sistema, después deniegan todo el tráfico excepto el que se da desde el localhost (la misma máquina que está ejecutando el servidor). Después de esto abrimos los puertos, el número 22 por el que entra el ssh, el número 80 por el que se da el acceso al servidor y por último (este puerto no viene en el guión lo he investigado yo por mi cuenta ya que me daba error al intentar acceder al servidor mediatne https) el puerto 443 que se encarga de las transferencias htttps entre el servidor y el cliente. Para ser más concretos muestro el guión en la captura 2.

![Captura 2](http://imgur.com/Pol4oWW.jpg "Captura 2")
Captura 2.

Una vez hemos hecho este script y lo hemos guardado en un archivo .sh para poder ejecutarlo lo que nos queda por hacer es que se ejecute cuando se inicie el sistema de forma automática. Para hacer esto hay muchas formas, en mi caso lo que he hecho ha sido añadirlo a "init.d" que es una carpeta en la que se encuentran los principales scripts que se ejecutan para el arranque del sistema. Para incluir un script en init.d debemos seguir los siguientes pasos siendo root en el sistema:

1º Movemos el script a la carpeta con la orden `sudo mv /home/usuario/script.sh /etc/init.d` sustituyendo la ruta inicial por la que cada uno tenga.
2º Le damos permisos de ejecución al script con la orden `sudo chmod +x /etc/init.d/script.sh`.
3º Hacemos una actualización del archivo update-rc.d para que cuando se inicie el sistema ejecute el script con la orden `sudo update-rc.d script.sh defaults`.

Una vez hecho esto reiniciamos el sistema, si arranca sin dar ningún error significa que el script está funcionando y sólo tendremos que tocar algunos parámetros sobre el para probar su funcionamiento.Un ejemplo de su funcionamiento es el que muestro en la captura 3. Intento hacer ping y no me lo permite ya que en iptables no tengo permitido que pase ningún paquete, incluidos los ICMP. En cambio, ejecuto curl y si que me saca lo que tiene el archivo index.html del servidor, con esto queda demostrado su funcionamiento de forma correcta.

![Captura 3](http://imgur.com/qAhhBZN.jpg "Captura 3")
Captura 3.


