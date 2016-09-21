# Servidores secuenciales con el API de sockets

### Material de soporte
-  On-line man pages: socket(2), socket(7), send(2), recv(2), read(2), write(2), setsockopt(2), fcntl(2), select(2), tcp(7), ip(7).
-  Guide to using sockets by Brian "Beej" Hall
-  Tcpdump manual
-  Chapters 6, 7 y 8 of "Linux Socket Programming" by Sean Walton, Sams Publishing Co. 2001

### Prácticas con sockets
Descripción de las prácticas de sockets Las prácticas de sockets se dividen en tres partes:
1. Servidores secuenciales (cliente y servidor de eco, opciones de sockets, análisis con tcpdump, servidores de ficheros)
2. Servidores concurrentes (procesos, hilos).
3. Entrada/Salida I/O (manejadores de señales, mecanismos de polling, select)

## Servidores secuenciales 

Los servidores secuenciales son los servidores más sencillos, generalmente los encontramos en servicios que se prestan por UDP (con sockets de tipo SOCK_DGRAM), pero también los podemos encontrar en servicios TCP (con sockets de tipo SOCK_STREAM).
Puedes refrescar la diferencia entre pasivos y activos así como las llamadas al API aquí [introducción a sockets](https://gitlab.pervasive.it.uc3m.es/aptel/intro-sockets). El ciclo de vida de las aplicaciones que utilizan sockets pasivos es el siguiente:

1. crear el socket (socket),
2. configurar el socket (estructura sockaddr_in, función getaddrinfo),
3. asociarlo con un puerto (bind),
4. convertir el socket en un socket pasivo para prepararlo para recibir conexiones entrantes (listen),
5. extraer una de las conexiones entrantes en espera y asociarla a un nuevo socket (accept),
6. lectura/escritura (read/write) o recibir/enviar (recv/send),
7. cierre del socket y liberación de la conexión (close),
8. volver al estado de espera de conexiones.

<img src="https://gitlab.pervasive.it.uc3m.es/aptel/intro-sockets/raw/master/overview_of_system_calls_used_with_stream_sockets.png" width="500px">

En esta práctica dispone de todos los ficheros necesarios para probar un servidor secuencial (se encuentra el código de un servidor de eco secuencial, y de un cliente de eco). Para ello, use el siguiente comando:

 ```
 git -c http.sslVerify=false clone https://gitlab.pervasive.it.uc3m.es/aptel/sockets1_sequential_servers.git
 ```

 Encontrarás dos ficheros: El primero es EchoServer_seq.c, que implementa el servidor. El segundo es EchoClient.c que implementa el cliente. El protocolo que implementan es ECHO sobre TCP/IP. En dicho servicio, el cliente de echo se conecta a un servidor de echo que copia de vuelta todo lo enviado por el cliente.
 
 Actividades
 
 1 Compila y ejecuta los ejemplos
 
 > En la explicación de la práctica se habla del puerto 8xxx, esto indica que debéis utilizar como puerto el resultado de sumarle a 8000 los tres últimos números de la dirección IP de la máquina en la que ejecuta el servidor. De esta forma, evitamos interferencias entre las prácticas realizadas por los diferentes grupos.
 
 Compile:
```
make clean
make
```

Ejecute el servidor en el puerto 8xxx:

```
~>./EchoServer_seq 8xxx
```

y en otra ventana, ejecute el cliente:
```
./EchoClient <server host> 8xxx
```
y observe su comportamiento.

> El parámetro `server host` es el nombre de máquina (hostname) o la dirección IP de la máquina a la que se quiere conectar.
Los nombres de máquina o (hostnames) son alias o nombres familiares para humanos que corresponden a una dirección IP de un dispositivo conectado a la red (la dirección sería mucho más complicada de recordar). Estos alias son traducidos a direcciones IP usando el servicio de DNS - Domain Name System (DNS) resolver.
Mira la etiqueta de tu ordenador. Encontrarás algo como un nombre digamos it001 que significa que el nombre de la máquina es it001.lab.it.uc3m.es. Justo debajo verás la dirección IP.
 