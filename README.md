para utilizar el script (desde un sistema UNIX, al que nos referiremos como C&C), rellenar el fichero attackers.txt con las IPs y credenciales de los sistemas atacantes (también UNIX, con acceso SSH) y el fichero NTP_Servers.txt con las IP de los servidores NTP abiertos (que responden a la petición "monlist"). El directorio bittwist-linux-2.0 debe tener dentro el código compilado de Bittwist*.

*Los atacantes deberán tener la misma arquitectura (32 o 64 bits). Por defecto, funcionará para 32 bits. Si se quieren utilizar atacantes de 64 bits, se deberá descargar el código de Bittwist (http://bittwist.sourceforge.net/) y compilarlo (make) en un sistema con dicha arquitectura, sin necesidad de instalarlo (make install). Las arquitecturas de la víctima y de los servidores no son relevantes.

Llamando al script con la opción '-h' se imprimirá la ayuda para utilizar el mismo:

Uso:
sudo python ddos-NTP.py <ip> [-n][-f]

  ip: IP del objetivo
-n: Enviar 600 ntpdate a cada NTP para amplificar el ataque (lento)
-f: Comenzar ataque automáticamente
Funcionamiento del script:
Se parsean los dos ficheros para obtener todos los datos necesarios.

Si hay un nuevo target se establece un tcpdump esperando a un paquete con IP origen la de la máquina y puerto 123. La captura se guardará como “NTP-Attack-Model.pcap”. Inmediatamente después se realiza una petición "monlist" al servidor NTP, provocando la captura de la petición (la respuesta del servidor no es importante). Este procedimiento se realizará por cada uno de los servidores NTP. Tras ello se hace spoofing de los paquetes generados en el paso anterior mediante Bittwist, poniendo como IP origen la de la víctima. Estos nuevos pcap se guardarán como “NTP-Attack-[IP_servidor].pcap”.

Si se ha puesto la opción ‘-d’, se realiza el paso 2 cambiando la petición "monlist" por una de tiempo. A partir de esos pcap, se envían 600 peticiones con IPs distintas a cada uno de los servidores NTP. Este paso es muy lento, tardará unos 60 segundos por cada servidor.

Por cada uno de los sistemas atacantes, se accede mediante SSH, se les copia por SCP un fichero comprimido con el código fuente de Bittwist (previamente compilado para una determinada arquitectura) y los pcap generados en el paso 2. Una vez copiado, se descomprime y se instala Bittwist en la máquina vulnerada.

[Directamente con la opción ‘-f’ o al pulsar Intro] Se genera un hilo por cada uno de los atacantes. En cada uno de esos hilos se vuelve a acceder por SSH al atacante correspondiente y se ejecuta Bittwist en modo inundación [‘-l 0’] a partir del pcap correspondiente (los servidores serán asignados mediante roundrobin a los atacantes, ya que de este modo se consigue un reparto equitativo).

Si se pulsa la tecla Intro, se vuelve a acceder por SSH a cada uno de los atacantes para parar la ejecución del proceso Bittwist, desinstalarlo y borrar todos los ficheros copiados previamente. Una vez realizado en todos (satisfactoriamente o no), el proceso del script se destruye y finaliza la ejecución del ataque.

Además, en todo momento el script comprueba que el ataque esté realizándose de una forma satisfactoria. En el momento en el que todos los atacantes hayan fallado (porque no se pueda instalar Bittwist o porque durante el ataque no respondan a una conexión SSH de prueba) el ataque es cancelado automáticamente.