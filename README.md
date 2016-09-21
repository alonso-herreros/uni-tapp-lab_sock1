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

En esta práctica dispone de todos los ficheros necesarios para probar un servidor secuencial. Para ello, use el siguiente comando:
 ```
 git -c http.sslVerify=false clone https://gitlab.pervasive.it.uc3m.es/distributed-computing-assignements/2-app-engine-servlet-jsp.git
 ```
 