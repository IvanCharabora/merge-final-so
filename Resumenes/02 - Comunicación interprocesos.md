[Volver](/README.md)

<h1>Comunicación interprocesos</h1>
La comunicación entre procesos sirve para compartir información, mejorar la velocidad de procesamiento, modularizar.
Se puede lograr IPC vía memoria compartida (se establece un área compartida donde ambos pueden leer/escribir), algún otro recurso compartido (archivo, base de datos, etc) o pasaje de mensajes.

![ipc_formas](/Resumenes/public/ipc_formas.png)

Entrando más en detale en pasaje de mensajes: los mensajes pueden tener una longitud fija (más facil pero la programación de tareas más tediosa) o variable (sistema más complejo pero más agilidad a la hora de programar tareas). 
Para que dos procesos se comuniquen debe existir un link de comunicación entre ellos. Para esto se debe tener en cuenta el tipo de comunicación que se desea ofrecer: 
* Direccionamiento: 
    * Conexión directa: cada proceso debe explicitar el nombre del proceso con el que desea comunicarse. Por ejemplo send(P, message) o receive(Q, message).
    * Comunicación indirecta: los mensajes son enviados a buzones o puertos. Cada puerto y buzón tiene un identificador único. Los procesos no necesitan conocer el nombre del proceso con el que se comunican. Por ejemplo send(port, message) o receive(port, message).
* Sincronización: 
    * Bloqueante: el proceso que envía un mensaje se bloquea hasta que el mensaje es recibido. El proceso que recibe el mensaje se bloquea hasta que el mensaje es recibido.
    * No bloqueante: el emisor envía algo que el receptor va a recibir en algún momento. Requiere algún mecanismo adicional para saber si el mensaje llegó. Libera al emisor para realizar otras tareas.
* Buffering: los mensajes intercambiados se almacenan en una cola temporal. Puede ser de capacidad 0 (no puede haber mensajes en espera), capacidad acotada (si la cola no está llena se puede envíar mensaje y si no se bloquea) o capacidad infinita (nunca se bloquea el remitente).



<h2>File descriptors</h2>
Representan instancias de archivos abiertos. Concretamente, son índices de una tabla que indica archivos abiertos por el proceso. Cada proceso viene con su propia tabla usada por el kernel para referenciar a los archivos abiertos. Cada emtrada apunta a un archivo.

![file_descriptors](/Resumenes/public/file_descriptors.png)
//global file table referencia a todos los archivos abiertos

Los file descriptors se heredan en el fork. Podemos leer y escribir en un fd con read y write. 

Ejemplo: echo “hola” > archivo.txt. El > hace que el stdout se redirija a un archivo.txt. Para esto abre archivo.txt y hace que la entrada del stdout en la tabla de file descriptor apunte a él. Esto se puede hacer con int dup2(oldfd, newfd).

![ejemplo_fd](/Resumenes/public/ejemplo_fd.png)

<br>

<h2>Pipes</h2>
Un pipe es un canal que provee una de las formas más simples de comunicación entre dos procesos aunque tienen sus limitaciones. Se representa como un archivo temporal y anónimo que se aloja en memoria y actúa como un buffer para leer y escribir de manera secuencial. Cosas a tener en cuenta: 

* ¿El pipe permite comunicación bidireccional o unidireccional?
Si es bidireccional, es half-duplex (para mandar información se debe esperar a que el otro extremo del pipe termina de hacerlo) o full-duplex (la información puede viajar de un lado a otro y viceversa simultaneamente).
* ¿Debe existir alguna relación entre los procesos que se están comunicando (por ejemplo,padre-hijo)?
* ¿Los procesos se van a poder comunicar dentro de una red o tienen que estar en la misma maquina?


![pipes](/Resumenes/public/pipes.png)
![pipes2](/Resumenes/public/pipes2.png)	
//La pipe se crea vía pipe(pipefd[2]) donde pipefd[0] es un fd que apunta a donde se lee y [1] a donde se escribe

//Si hago close fd[0] en el padre y close fd[1] en el hijo estoy cerrando la lectura del parent y la escritura del hijo.

Por defecto el write es bloqueante buffereado (si hay lugar la función retorna que se completó, si no se bloquea hasta escribir todo), por defecto el read es bloqueante buffereado (si hay algo para leer retorna, si no se bloquea hasta que haya algo para leer).


Los pipes pueden ser:
* Ordinary pipes: Permiten que dos procesos se comuniquen en modo productor-consumidor: El productor escribe en un extremo del pipe (extremo de escritura) y el consumidor lee desde el otro
extremo (extremo de lectura). Estos pipes son unidireccionales
* Named pipes: Proveen comunicación bidireccional y no necesitan que los procesos estén relacionados
<br>
<h2>Sockets</h2>
Los sockets son los extremos de una comunicación que usan dos procesos para comunicarse a través de una red. Cada socket está identificado por una dirección IP y un número de puerto. En general, en este tipo de links, se usa una arquitecutra cliente-servidor: El servidor espera
a que un cliente haga un pedido y, una vez que lo recibe, acepta la conexión del socket del cliente
para completar la conexión.

![socket](/Resumenes/public/socket.png)
* socket(): crea un socket y devuelve un file descriptor que lo representa.
* bind(): asocia el socket a una dirección IP y un puerto.
* listen(): pone al socket en modo de escucha.
* accept(): extrae de la cola una solicitud de conexión y establece la comunicación entre los sockets. Se bloquea en caso de no existir conexiones pendientes.
* connect(): establece la conexión con el socket del servidor.

Se comunican via send y recv. Por defecto son bloqueantes pero se puede hacer que no.