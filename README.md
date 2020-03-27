# Cómo acceder a un ordenador mediante ssh, cuando éste se ha protegido de forma que sea necesaria la doble verificación mediante una llave U2F.

Cuando hemos configurado el uso de la doble verificación a la hora de acceder a un ordenador o de ejecutar el comando sudo,nos encontramos con el problema de que al acceder a ordenador vía ssh, se nos va a solicitar que pulsemos el botón de la llave U2F, pero en el ordenador al que queremos acceder. Como es lógico, esta conexión la estaremos realizando desde un ordenador remoto, por lo que no nos será posible conectar la llave ni pulsar el botón.

Esta situación se nos plantea cuando [hemos configurado la llave U2F](https://github.com/fbuenonet/Hyperfido/edit/master/README.md) en el archivo */etc/pam.d/common-auth* lo que afecta a todos los servicios relacionados con el acceso al equipo y con las ejecución de comandos mediante sudo. Básicamente podemos decir que si se necesita introducir una contraseña, será necesaria la doble verificación conectando la llave U2F y punldando el botón.

Podríamos hacer que la solicitud de la doble verificación se pidiera para todos los serivicios excepto para sshd. Lo que habría que hacer es no poner la línea que solicta la doble verificación en el archivo */etc/pam.d/common-auth*, ya que éste es llamado con un "include" desde todos los archivos de servicios, e insertar esa línea en todos aquellos de configuración de los servicios, excepto en aquellos que no queramos que usen la doble verifcación. Esto plantea un problema de inseguridad, ya que anulamos la doble verificación ante ciertos servicios a cambio de una supuesta comodidad.

Pero es posible mantener la seguridad y además acceder de forma remota cómodamente y sin necesidad de teclear la contraseña. Para ello tenemos que crear un par de claves RSA, privada y pública, en el equipo desde el que nos vamos a concetar y tedremos que comunicar al equipo remoto cuál es nuestra clave pública y a qué usuario pertenece. El proceso es muy sencillo:

### 1. Creación del par de claves. <h3>

Antes de nada, debemos tener instalado en el ordenador local un software que nos permita emular un terminal remoto. Para usuarios de Linux, basta usar la consola. Los usuarios de Windows deberán usar un software como [PuTTY](https://putty.org/) y en el caso de Android, pueden usar la app [ConnectBot](https://play.google.com/store/apps/details?id=org.connectbot)

Para quien no sepa cómo generar un par claves desde PuTTY, puede consultar el tutorial que ofrece [Microsoft](https://docs.microsoft.com/es-es/azure/virtual-machines/linux/ssh-from-windows) que está suficientemente claro. En el caso de querer generar el par de claves desde ConnectBot, no se necesita demasiada explicación, porque el proceso es muy simple, pero sí comentaré que hay que seguir 3 pasos imprescindibles:

        a) Crear la clave. Solo hay que usar la opción destinada a ello
        b) Exportar a un archivo la clave pública, llamándolo: nombre_del_usuario.pub
        c) Enviar al ordenador remoto la clave pública
        
 Si nos encontramos en un ordenador Linux, podremos crear el par de claves RSA tecleando el siguiente comando:
 
        ssh-keygen -t rsa
 
Se nos pedirá el nombre del usuario para el que se genera la clave y a continuación la contraseña maestra con la que protegeremos la clave. Se podría dejar en blanco si vamos a usar la clave desde un dispositivo que disponga de una protección fuerte de su contenido, en caso contrario, habrá que usar necesariamente una contraseña maestra.

Las claves que acabamos de generar, se habrán huardado en */home/usuario/.ssh* y estarán contenidas en *id_rsa* (clave privada) y *usuario.pub* (clave pública, donde usuario corresponde al nombre del usuario que hayamos indicado).

### 2. Envío de la clave pública al ordenador remoto <h3>

Vamos a ver cómo enviar el archivo con la clave pública al ordenador remoto. Primero lo haremos desde el ordenador Línux y después veremos cómo hacerlo desde Android. Para ello teclearemos:

        scp usuario.pub usuario@dominio.com:/home/usuario/.ssh
        (Sustituiremos "usuario" por el nombre de usuario y "dominio.com" por el nombre del dominio
        que identifica a la máquina remota o su dirección IP).
        
Para enviar el archivo con la clave pública desde un dispositivo Android hasta el ordenador remoto, hay muchas posiblidade, por lo que mejor que contarlas aquí, creo que es mejor que veas este artículo, donde ya están descritas muchas de ellas y así podrás escoger la que más te convenga dependiendo de si ya tienes instalada alguna de las app o si es que te resulta familar cualquiera de los métodos descritos. Puedes leer el artículo completo desde [aquí](https://www.xatakandroid.com/tutoriales/como-enviar-archivos-de-android-al-pc-y-viceversa-todos-los-metodos-disponibles)

      Recuerda que el archivo que debes copiar es el denominado usuario.pub
      y que debes guardarlo en /home/usuario/.ssh
      
 ### 3. Identificar al usuario y su clave pública en el ordenador remoto
 
Antes de ver cómo actuar para realizar la identificación del usuario y su clave en el ordenador remoto, voy a realizar una reflexión sobre todo el proceso. 

Todo este proceso parte del supuesto de que estamos ante un ordenador (posiblemente un servidor) remoto al que queremos acceder vía ssh y sin la necesidad de teclear la contraseña, cuando hemos activado la doble verificación usando una llave U2F. Pero, ¿realmenmte esto tiene sentido, cuando se trata de un servidor al que no podemos acceder físicamente para conectar una llave en el puerto USB y luego pulsar el botón de la llave cuando sea necesario? Evidentemente esta es una situación que jamás se nos va a plantear, por lo que únicamente es válida para un ordenador propio y al que nos conectaremos desde otra habitación de la casa o, si acaso, desde el exterior a través de una conexión desde Internet. En cualquier caso, siempre vamos a partir de una configuración hecha en local, que siempre es más cómodo. Por eso, aunque es posible finalizar la configuración desde el móvil o un ordenador situado en cualquier parte, voy a finalizar al explicación del proceso pensando en que estamos sentados en el ordenador que siempre hemos denominado como "remoto" y sabiendo que todo lo explicado hasta el momento es válido y muy interesante de conocer, porque nunca sabemos cuándo lo vamos a necesitar.

Por repasar un poco lo hecho hasta el momento, ya tenemos el par de claves RSA creadas, ya sea con el móvil o en el propio ordenador (que seguiré llamando remoto). También tenemos en el ordenador remoto la clave pública y la privada si la hemos creado en este ordenador y ahora ya sólo nos queda el último paso, que es identificar al usuario y su clave pública en este ordenador.

### 4. Identificación del usuario y su clave pública en el ordenador remoto <h4>
 
Lo primero que debemos hacer es acceder a la carpeta .ssh. Para ello tecleamos en el terminal: *cd ssh* Doy por hecho que estamos en la carpeta home del usuario, de lo contrario tecleamos *cd* y pulsamos Enter.

Una vez en la carpeta .ssh, debemos crear, en caso de que no exista, el archivo *authorized_key* y a continuación añadir la clave pública a dicho archivo. Para ello tecleamos:

      touch authorized_keys
      cat usuario.pub >> authorized_keys
      
Con esto ya hemos finalizado el proceso y podremos acceedr al ordenadore remoto desde el móvil para apagarlo o relizar cualquier otra operación. Realizarlo es mucho más corto que todo lo que he escrito aquí, pero creo que merecía la pena desarrollarlo de forma tan extensa para comprender mejor todas las posiblidades de hacerlo.
