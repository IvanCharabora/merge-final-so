<h1>01, 02, 03</h1>
<h2>Diferencia entre mode switch y context switch</h2>
A CPU has two primary “modes” (Some CPU’s may have others). They are kernel mode and user mode. A mode switch is performed by a trap or syscall shifting the CPU from user mode to kernel mode and then back to user mode when the syscall is completed. This is done when some code needs to access privileged resources within the control of the kernel.

A context switch is the CPU saving of the state of a process and restoring the state of another process and starting the execution of the next process. A context switch can be performed from one application to another running on the machine (multi-tasking) or from one thread to another within the same process. However, the context switch between threads keeps the memory intact, as each thread in a process has access to that process's memory.

<h2>fork vs vfork</h2>
En el caso de fork() el proceso hijo apunta a una copia de las páginas de memoria del padre. Aunque en realidad se utiliza la técnica copy-on-write, copiando las páginas sólo cuando sea necesario, para optimizar el uso de memoria.

Por el otro lado vfork() suspende al padre y le da al hijo una copia de las páginas, sin copy-on-write, es decir, cualquier modificación del hijo va a ser vista por el padre al finalizar y retornar el control. Esto se debe a que vfork tiene el mismo espacio de memoria que el padre.

<h2>Procesos y threads</h2>
Proceso: Es un programa puesto a ejecutar. Puedo tener multiples procesos ejecutando simultaneamente para un mismo programa. Es la unidad de trabajo en un sistema.
Thread: Es un unidad basica de utilizacion del CPU, el cual contiene un thread id, un program counter, un set de registros y un stack. Comparte a su vez con otros threads que pertenecen al mismo proceso la seccion de codigo, de datos y otros recursos del OS como los files abiertos.

Como se ve, un proceso no comparte su espacio de memoria con otros (excepto quizas bibliotecas compartidas x ejemplo, pero no stack, heap, codigo) a diferencia del thread que si comparte heap, codigo pero no stack tampoco.

<h2>Agregar memoria y procesador a un sistema</h2>
Caso Procesador: Agregar un procesador mas tiene sentido si los procesos se encuentran normalmente en los estados READY y RUNNING. Esto implica que nunca llegan a bloquearse porque probablemente no les da el quantum y son desalojados antes de esto. Al agregar un procesador, se pueden tener mas procesos corriendo a la vez, osea se mejora el throughput y potencialmente pueden llegar a pasar a BLOCKED mas frecuentemente. Habria que chequear esto ultimo analizando la *carga del sistema*, que seria la cantidad de procesos en READY.

Caso Memoria: Agregar memoria tiene sentido si los procesos estan mayormente BLOCKED porque se esta produciendo trashing, es decir, la memoria esta ocupada por todos los procesos. Esto provoca que cuando un proceso quiere acceder a una porcion de memoria que no esta cargada en memoria, se produzca un page fault, que lleva a un context switch (es caro), y el SO tiene que encargarse de swappear una pagina de la memoria con la del disco que necesita el proceso. Esto ultimo es lento y provoca que el proceso pase a BLOCKED.

<h2>Inversión de prioridades</h2>
Hay 3 tareas, una de prioridad alta H, una de prioridad media M y una de prioridad baja L. H y L comparten un recurso, que M no necesita. Supongamos que L está corriendo y M lo saca, por tener mayor prioridad. Entonces L no suelta el recurso (porque M no lo necesita) y cuando H saque a M no va a tener el recurso. 
Afecta a los schedulers con preemption (prioridades).

<h2>Scheduling con muchos procesadores</h2>
El caché es de vital importancia para el rendimiento de los programas. Cada procesador tiene su cache, por lo que, si la política de scheduling hace pasar un proceso a otro procesador, éste llega con el caché vacío, tardando más de lo que tardaría si se hubiese ejecutado en el mismo procesador de antes. Para evitar esto se usa el concepto de afinidad al procesador: tratar de usar el mismo procesador, aunque se tarde más en obtenerlo. También se puede intentar distribuir la carga entre todos los procesadores, a esto se lo llama push y pull migration. En push migration un procesador está sobrecargado en su cola de procesos, y le “pushea” a otro procesador alguno de esos procesos. En pull migration, un CPU ve que tiene espacio libre en su cola y “pullea” de otro procesador un proceso.


<h1>4</h1>
Acceso a un contador que se desea incrementar mediante un semáforo binario: No es correcta. Las operaciones de los semáforos se implementan mediante system calls, lo cual implica que se debe realizar un cambio de contexto cada vez que son realizadas. Esto tiene un overhead que no se justifica para un acceso exclusivo de tan corto tiempo como lo es incrementar un contador.

<h2>Diferencias entre spinlock y semáforo</h2>

* Los spinlocks se usan solamente para exclusión mutua, mientras que los semáforos se pueden usar para exclusión o como semáforos no binarios. 
* Un spinlock es un mecanismo de sincronización de bajo nivel, y un semáforo es un mecanismo de señales.
* Los semáforos puede permitir que más de un proceso tenga acceso a la sección crítica al mismo tiempo, pero los spin locks sólo permiten acceso a CRIT de a un proceso a la vez.
* Los spinlocks funcionan para un sólo proceso, mientras que los semáforos se pueden usar para sincronizar varios procesos.
* Una diferencia muy importante es que los spinlocks hacen busy waiting, gastando mucho procesador. En cambio, en el caso de los semáforos, si un proceso espera por uno se va a dormir, para ser despertado más adelante, por lo que no hay desperdicio de tiempo de procesamiento ni de recursos. 
* Los spinlocks son eficientes porque se bloquean por un plazo corto de tiempo, pero los semáforos se bloquean por más tiempo. Para acceder a su estructura de control, usan spinlocks.

Teniendo en cuenta todo esto, si se tiene un sistema uniprocesador, conviene usar semáforos, ya que no mantienen al procesador ocupado mientras se espera por el lock, mientras que un spinlock mantendría al procesador sin poder realizar nada más hasta obtener el lock. 
En general, si se sabe que la espera va a ser corta (es decir, la sección crítica es poco compleja), conviene usar spinlocks. 


<h2>Que pasa cuando se hace un signal</h2>
Un semáforo tiene un valor y una lista de punteros a PCB que representa los procesos esperando por él. Cuando se hace un signal(), se aumenta en 1 el valor del semáforo, y si este es <= 0, se saca el proceso de la lista y hay que “despertar al proceso”, esto es, ponerlo listo para ejecutar (estado ready) y agregarlo a la cola de proceso esperando.

<h2>Algoritmo de Peterson</h2>
El algoritmo de Peterson es un algoritmo que usa dos variables (flag y turn) para sincronizar el uso de un recurso entre dos procesos (resuelve el problema de la sección crítica). Se restringe a dos procesos que alternan su ejecución entre la sección crítica y las demás secciones. Los dos proceso deben compartir dos datos: un int turn (indica de quién es el turno de entrar a CRIT, si turn = i, entonces P_i puede entrar a CRIT) y un bool flag[2] (este arreglo se usa para indicar si un proceso está listo para entrar a su sección crítica).

Para entrar a la sección crítica, un proceso P_1 setea flag[i] en true y pone turn en el valor 0, así, si el otro proceso quiere entrar a la sección crítica, puede hacerlo. Si ambos procesos quieren entrar al mismo tiempo a CRIT, turn va a ser 1 y 0 casi al mismo tiempo. Sólo una de estas asignaciones va a durar (la otra se escribe inmediatamente). 

<h1>7</h1>
<h2>que beneficio tengo por escribir los datos secuencialmente en CD/DVD por ejemplo?</h2>
performance, algoritmo sencillo, menos metadatos porque puedo decir donde empieza y termina un file

<h2>SAN</h2>
Es una red privada que conecta multiples servidores y unidades de almacenamiento. Permite tener multiples servidores y almacenamientos conectados a la vez, y el almacenamiento para un server se puede reservar dinamicamente. Por ejemplo, si un server esta con poco almacenamiento de disco, SAN puede reservarle mas a el. SAN le permite a clusters de servers compartir el mismo almacenamiento.

Es una forma de hacer parecer que dispositivos de almacenamiento que están conectados a una red parecen estar conectados directamente a la máquina. Se acceden como si fueran archivos locales. 

<h1>8</h1>
<h2>Estructuras afectadas al hacer rm de un archivo en linux</h2>

* en EXT2 se marca como unused al inode, pero se dejan intactos los pointers
* en EXT3 se zeroean los pointers por el SO
* se elimina el inodo de la lista del directory
* se decrementa en 1 el link count de hard links
* si el count llego a 0 entonces se marca al inode como eliminado
* se marcan en el bitmap del block group a los blocks de este inodo como libres

<h2>Ext2 vs Ext3</h2>
Ext2
the disk partition is divided into groups of blocks, irrespective of where the disk cylinder boundaries fall.

The first block is the superblock. It contains information about the layout of the file system, including the number of i-nodes, the number of disk blocks, and the start of the list of free disk blocks (typically a few hundred entries). Next comes the group descriptor, which contains information about the location of the bitmaps, the number of free blocks and i-nodes in the group, and the number of directories in the group. This information is important since ext2 attempts to spread directories evenly over the disk.

Two bitmaps are used to keep track of the free blocks and free i-nodes, respectiv ely, a choice inherited from the MINIX 1 file system (and in contrast to most UNIX file systems, which use a free list).

Following the superblock are the i-nodes themselves. They are numbered from 1 up to some maximum. Each i-node is 128 bytes long and describes exactly one file. An i-node contains accounting information (including all the information returned by stat, which simply takes it from the i-node), as well as enough information to locate all the disk blocks that hold the file’s data. Following the i-nodes are the data blocks. All the files and directories data are stored here. If a file or directory consists of more than one block, the blocks need not be contiguous on the disk. In fact, the blocks of a large file are likely to be spread all over the disk. I-nodes corresponding to directories are dispersed throughout the disk block groups.

Ext3

La diferencia principal que tiene con Ext2 es que se le agrega journaling

<h2>Porque se separa en block groups?</h2>
Los block groups contienen informacion del bloque y varios inodos. La ventaja que aportan es que los files guardados en el mismo block group pueden ser accedidos con menor seek time en promedio de disco.


<h2>ls vs ls -l</h2>
ls solo debe acceder al inodo del directorio para listar los nombres de los archivos. ls -l, en cambio, accede al inodo de cada archivo para listar sus metadatos.


<h2>ACLs en FAT, NTFS, EXT2</h2>
ACL (Access Control List): Se asocia a cada objeto una lista ordenada que contiene todos los dominios (conjuto de pares objeto-permiso) que pueden acceder al objeto y cómo. Cada archivo tiene asociada una ACL. 

FAT: solo se pueden marcar archivos como “SYSTEM”

Ext2: dentro de la estructura del inodo hay un atributo de permisos, un user id y un group id. Los permisos pueden ser read, write y/o execute. 

NTFS: A cada archivo o carpeta se le asigna un descriptor de seguridad que define su dueño y contiene 2 ACLs. La primera es se llama DAC list y define qué tipo de interacciones están permitidas o prohibidas para cada usuario o grupo de usuarios. Windows Vista agrega información MAC a la DACL. La segunda ACL, llamada System ACL, define qué interacciones con el archivo o carpeta deben ser auditadas y cuáles van a ser loggeadas cuando la actividad sea exitosa, falla o ambas. 


<h1>9</h1>
<h2>explicar diferencias entre sistema paralelo y distribuidos</h2>
Un sistema paralelo tiene un equipo con múltiples procesadores, que se comunican entre sí a través de memoria, bus y clock compartidos. Por otro lado, un sistema distribuido es un conjunto de nodos conectados a una red, es decir, se tienen varias computadoras autónomas que interactúan entre sí, las cuales se abstraen como un solo sistema a la vista del usuario. Las máquinas no comparten clock, memoria, ni bus de datos en un sistema distribuido.


<h1>10</h1>
Firma digital: La idea es que queremos enviar un mensaje grande, tipo documento, y que la otra persona tenga certeza de quien es el autor del mensaje. Podemos hacer esto con RSA, pero es muy costoso el algoritmo sobre un documento entero. Entonces la idea es que en lugar de encriptar todo el documento, se encripta un digest del mismo, que seria un hash. Luego se envia el conjunto (hash encriptado, documento original, clave publica) El que lo reciba puede desencriptar el hash con la publica, hashear el documento y comparar que estos 2 son iguales. Si esto ocurre, entonces el documento es el original del que lo envio, sino no.

Certificado digital: es una forma de compartir una clave publica certificada que identifica una entidad que es dueña de la misma.

Autoridad certificante: son los que emiten los certificados digitales y son una entidad confiable.

<h2>Bit executed</h2>
Con el bit NX (No eXecute) podemos evitar que se ejecute shell code luego de que se hace un buffer overflow inyectandolo
