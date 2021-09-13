[English version] (README_EN.md)

# Servidores secuenciales con el API de sockets

### Material de soporte
-  On-line man pages: socket(2), socket(7), send(2), recv(2), read(2), write(2), setsockopt(2), fcntl(2), select(2), tcp(7), ip(7).
-  Guide to using sockets by Brian "Beej" Hall
-  Tcpdump manual
-  Chapters 6, 7 y 8 of "Linux Socket Programming" by Sean Walton, Sams Publishing Co. 2001
-  Fichero cheat sheet en este proyecto

### Prácticas con sockets
Descripción de las prácticas de sockets Las prácticas de sockets se dividen en tres partes:
1. Servidores secuenciales (cliente y servidor de eco, opciones de sockets, análisis con tcpdump, servidores de ficheros)
2. Servidores concurrentes (procesos, hilos).
3. Entrada/Salida I/O (manejadores de señales, mecanismos de polling, select)

El **objetivo** de estas prácticas es entender y resolver las preguntas que se formulan a continuación. No es necesario entregar las respuestas, a menos que tengáis interés en tener retroalimentación del profesor (por correo electrónico), de cara al examen de prácticas final. 

## Servidores secuenciales 

Los servidores secuenciales son los servidores más sencillos, generalmente los encontramos en servicios que se prestan por UDP (con sockets de tipo SOCK_DGRAM), pero también los podemos encontrar en servicios TCP (con sockets de tipo SOCK_STREAM).

Puedes refrescar la diferencia entre pasivos y activos así como las llamadas al API aquí [introducción a sockets](https://gitlab.gast.it.uc3m.es/aptel/intro-sockets). El ciclo de vida de las aplicaciones que utilizan sockets pasivos es el siguiente:

1. crear el socket (socket),
2. configurar el socket (estructura sockaddr_in, función getaddrinfo),
3. asociarlo con un puerto (bind),
4. convertir el socket en un socket pasivo para prepararlo para recibir conexiones entrantes (listen),
5. extraer una de las conexiones entrantes en espera y asociarla a un nuevo socket (accept),
6. lectura/escritura (read/write) o recibir/enviar (recv/send),
7. cierre del socket y liberación de la conexión (close),
8. volver al estado de espera de conexiones.

<img src="https://gitlab.gast.it.uc3m.es/aptel/intro-sockets/raw/master/overview_of_system_calls_used_with_stream_sockets.png" width="500px">

En esta práctica dispone de todos los ficheros necesarios para probar un servidor secuencial (se encuentra el código de un servidor de eco secuencial, y de un cliente de eco). Para ello, use el siguiente comando:

 ```
 git clone https://gitlab.gast.it.uc3m.es/aptel/sockets1_sequential_servers.git
 ```

 Encontrarás dos ficheros: El primero es `EchoServer_seq.c`, que implementa el servidor. El segundo es `EchoClient.c` que implementa el cliente. El protocolo que implementan es ECHO sobre TCP/IP. En dicho servicio, el cliente de echo se conecta a un servidor de echo que copia de vuelta todo lo enviado por el cliente.
 
## Actividades
 
### 1. Compila y ejecuta los ejemplos
 
 > En la explicación de la práctica se habla del puerto `8xxx`, esto indica que debéis utilizar como puerto el resultado de sumarle a `8000` los tres últimos números de la dirección IP de la máquina en la que ejecuta el servidor. De esta forma, evitamos interferencias entre las prácticas realizadas por los diferentes grupos.

Dentro de la carpeta descargada, entra en `sockets1_sequential_servers/psockets1`

```
cd sockets1_sequential_servers
cd psockets1
```

Compile:
```
make clean
make
```

Ejecute el servidor en el puerto 8xxx:

```
./EchoServer_seq 8xxx
```

y en otra ventana, ejecute el cliente:
```
./EchoClient <server host> 8xxx
```
y observe su comportamiento.

> El parámetro `server host` es el nombre de máquina (hostname) o la dirección IP de la máquina a la que se quiere conectar.
Los nombres de máquina o (hostnames) son alias o nombres familiares para humanos que corresponden a una dirección IP de un dispositivo conectado a la red (la dirección sería mucho más complicada de recordar). Estos alias son traducidos a direcciones IP usando el servicio de DNS - Domain Name System (DNS) resolver.
Mira la etiqueta de tu ordenador. Encontrarás algo como un nombre digamos it001 que significa que el nombre de la máquina es it001.lab.it.uc3m.es. Justo debajo verás la dirección IP.
 
Echa un vistazo al código del servidor en `EchoServer_seq.c`. Responde a las siguientes preguntas:
* ¿Qué variable almacena el socket pasivo?
* ¿Qué variable almacena el socket activo?
* ¿En qué línea obtenemos el socket activo?

> Para verificar que el servidor está escuchando apropiadamente ejecuta netstat de la siguiente forma (ejemplo-mira las líneas rojas – un * en la columna address significa todas las direcciones):
```
./EchoServer_seq 8765
Waiting for incomming connections at port 8765
Incomming connection from 127.0.0.1 remote port 52589
```

Asegurate que el servidor está escuchando en el puerto indicado:
```
netstat -ta
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address Foreign Address State
tcp 0 0 *:sunrpc *:* LISTEN
tcp 0 0 *:28017 *:* LISTEN
tcp 0 0 *:ssh *:* LISTEN
tcp 0 0 localhost:ipp *:* LISTEN
tcp 0 0 localhost:smtp *:* LISTEN
tcp 0 0 *:8765 *:* LISTEN
tcp 0 0 *:40645 *:* LISTEN
tcp 0 0 *:27017 *:* LISTEN
tcp 0 0 localhost:mysql *:* LISTEN
tcp 0 0 localhost:52589 localhost:8765 ESTABLISHED
tcp 0 0 localhost:8765 localhost:52589 ESTABLISHED
tcp6 0 0 [::]:sunrpc [::]:* LISTEN
tcp6 0 0 [::]:http [::]:* LISTEN
tcp6 0 0 [::]:ssh [::]:* LISTEN
tcp6 0 0 ip6-localhost:ipp [::]:* LISTEN
tcp6 0 0 ip6-localhost:smtp [::]:* LISTEN
tcp6 0 0 [::]:32769 [::]:* LISTEN
```

Vamos a observar las conexiones TCP con la herramienta tcpdump. Vamos a decirle que nos reporte todo el tráfico con origen o destino el puerto 8xxx y que se escuche en el interfaz del bucle local (loopback).

> Debes comprender que cada máquina tiene al menos dos interfaces de red: 
> - Los estándares de red IPv4 reservan el segmento 127.0.0.0/8 para loopback. Eso significa que cualquier paquete enviado a alguna de esas direcciones (127.0.0.1 a 127.255.255.254) será enviado de vuelta al origen.
> - La dirección IP real (en este caso dentro de la red de la universidad) que comienza por 163.117 (uc3m) debería ser algo como: 163.117.XXX.XXX

### 2. Ejecuta el siguiente comando en otra ventana:

```
sudo tcpdump -i lo port 8xxx
```

Presiona alguna tecla en el cliente para ver el eco. Vuelve a lanzar el cliente y observa el intercambio de tramas con tcpdump (establecimiento de conexión, envío de datos en ambos sentidos, asentimientos y fin de conexión)

> Ejemplo de uso de tcpdump que muestra un establecimiento de conexión
```
sudo tcpdump -i lo port 8888
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on lo, link-type EN10MB (Ethernet), capture size 65535 bytes
16:28:57.553866 IP localhost.39363 > localhost.8888: Flags [P.], seq 470686013:470686018, ack 1031632982, win 513, options [nop,nop,TS val 2951382 ecr 2948837], length 5
16:28:57.554210 IP localhost.8888 > localhost.39363: Flags [.], ack 5, win 512, options [nop,nop,TS val 2951382 ecr 2951382], length 0
16:28:57.554620 IP localhost.8888 > localhost.39363: Flags [P.], seq 1:6, ack 5, win 512, options [nop,nop,TS val 2951382 ecr 2951382], length 5
```

### 3. Lanza el cliente un par de veces y finalízalo con CTRL-D o CTRL-C. Ahora mata el servidor con CTRL-C ¿Qué se observa con el tcpdump? ¿Por qué? 

**Solucione el problema. No continúe sin solucionarlo (deberá editar el código fuente)**

> Puede ser útil observar el estado de TCP utilizando para ello el comando: netstat -tn o netstat -putan
```
netstat -tn
Active Internet connections (w/o servers)
Proto Recv-Q Send-Q Local Address Foreign Address State
tcp 0 0 127.0.0.1:39363 127.0.0.1:8888 ESTABLISHED
tcp 0 0 127.0.0.1:8888 127.0.0.1:39363 ESTABLISHED
```

**A partir de ahora vamos a lanzar los clientes en una máquina distinta**. Para ello en una ventana nueva ejecuta el comando slogin o ssh (para ejecutar comandos en otra máquina – uso ssh nombrehost o ssh dirIP). Necesitaremos parar y rearrancar el tcpdump, esta vez sin especificar la interfaz (eliminar el parámetro –i lo pero manteniendo el puerto), de tal forma que se monitoriza el tráfico por todas las interfaces de red salvo por el bucle local.

### 4. Vuelve a arrancar el servidor (ya modificado), arranca un cliente y mata el servidor sin matar al cliente ¿En qué estado se encuentra el servidor? Ahora, mata al cliente ¿Cambia el estado del servidor? ¿Qué ocurre al arrancar el servidor de nuevo (no esperes mucho tiempo)? ¿Por qué? **No continúe sin averiguarlo**. Espera unos pocos segundos y lanza de nuevo el servidor. 

### 5. Lanza ahora dos clientes de eco contra el mismo servidor (desde dos máquinas distintas) ¿Qué ocurre si después de lanzar el primer cliente, lanzamos el segundo? Envía información desde ambos clientes ¿Qué ocurre?

### 6. Mata al segundo cliente y después al primero. Debe de observar una trama de RESET ¿Por qué?

### 7. Cambia el tamaño de la cola de conexiones (backlog del socket) del servidor, definido por la constante BACKLOG, en el fichero EchoServer_seq.c, a 1 y recompila el código. Ahora lanza cuatro clientes de eco desde otra máquina. ¿Qué aprecia desde tcpdump? ¿Qué explicación se te ocurre?

Vamos a manipular las opciones del socket que utiliza el cliente utilizando setsockopt. Para más información sobre las opciones que pueden cambiarse, se sugiere consultar las páginas del manual ip(7), tcp(7), setsockopt(2), socket(7).

### 8. En primer lugar vamos a permitir enlazar un socket a un puerto que ya está en uso - activando la opción SO_REUSEADDR con la función setsockopt - (mientras no exista un socket pasivo en estado activo ya enlazado a él). Para ello, debes descomentar la línea de código apropiada en EchoServer_seq.c y recompilar el código. Observa el resultado de modificar esta opción revisando la pregunta 4.

### 9. Modifica ahora el tamaño de los segmentos enviados por el cliente, activando la opción TCP_MAXSEG con la función setsockopt:

```c
int maxseg = 88;
if (setsockopt(sockfd, SOL_TCP, TCP_MAXSEG, &maxseg,sizeof(maxseg))==-1)
{…}
```
> En ocasiones, dependiendo del kernel de linux, el valor mínimo de TCP_MAXSEG puede ser diferente. Además, en ocasiones dicho parámetro puede no afectar al interfaz loopback (localhost) pero si a interfaces físicos (eth0, eth1...) 

Recompila y comprueba este efecto en un cliente escribiendo textos más largos que la longitud especificada. ¿Es posible especificar MSS inferiores a 88?. 

Si no se puede, ¿cuál crees que es el motivo?

