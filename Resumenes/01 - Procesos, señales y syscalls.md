[Volver](/README.md)

<h1>Procesos, señales y syscalls</h1>
<h2>Procesos</h2>
Es una instancia de un programa en ejecución. A cada proceso se le asigna un identificador único llamado pid. Visto desde memoria un proceso contiene el stack del proceso, el área de texto (código de máquina del programa), el área de datos (donde se almacena el heap de memoria). Cada proceso tiene su propia CPU virtual, es decir cada proceso piensa que tiene una CPU propia.
La diferencia entre un programa y un proceso es que el programa es una entidad pasiva (como un archivo conteniendo una secuencia de instrucciones) mientras que el proceso es una entidad activa que tiene asociado una serie de recursos como un programa, entrada, salida, estado, etc.

Un proceso puede:
* Terminar (exit). Indica al SO que ya puede liberar los recursos asociados al proceso. Además indica un status de terminación que le es reportado al padre.
* Lanzar un proceso hijo (system, fork, exec). Fork hace una copia exacta del padre (devuelve el pid del hijo si estoy en el padre o 0 si estoy en el hijo). El hijo puede ser lanzado con un programa diferente al del padre (exec). El padre puede esperar a que el hijo termine (wait). Con todo esto se puede formar un arbol de procesos.
* Ejecutar en la CPU
* Hacer un system call
* Realizar E/S

Durante su ejecución un proceso puede cambiar de estado:
* New: El proceso está siendo creado.
* Ready: El proceso está listo para ejecutarse.
* Running: El proceso está ejecutándose.
* Waiting: El proceso está esperando a que suceda algo (por ejemplo que termine un proceso hijo o que se complete una operación de E/S).
* Terminated: El proceso terminó su ejecución.

Estos estados se van modificando de la siguiente manera:

![estados_procesos](/Resumenes/public/estados_procesos.png)

<h3>Arbol de procesos</h3>
Todos los procesos están organizados jerárquicamente como un árbol. Cuando el SO comienza, lanza un proceso que se suele llamar init. Por eso es importante la capacidad de lanzar un proceso hijo.

<h3>Mapa de memoria de un proceso</h3>

![memoriaProceso](/Resumenes/public/memoriaProcso.png)

A process is more than the program code, which is sometimes known as the text section. It also includes the current activity, as represented by the value of the program counter and the contents of the processor’s registers. 
A process generally also includes the process stack, which contains temporary data (such as function parameters, return addresses, and local variables), and a data section, which contains global variables. A process may also include a heap,which is memory that is dynamically allocated during process run time.

<h3>Scheduling en Procesos</h3>
Acordarse que solo un proceso a la vez puede estar en la CPU. Entonces hay que dejarlo un "ratito", a ese "ratito" lo llamamos quantum. En general, cuando se acaba el quantum le toca el turno al siguiente proceso (preemption). 

Scheduler: componente esencial del kernel. Su función es decidir a que proceso le corresponde ejecutar en cada momento. Hay varias formas de decidir esto. Que proceso elegir depende de la política de scheduling que se esté usando. 

Para cambiar el programa que se ejecuta en la CPU debemos:
* guardar los registros 
* guardar el IP
* si se trata de un programa nuevo, cargar el nuevo programa en memoria
* cargar los registros
* cargar el IP

A esto se le llama context switch. El IP y demás registros se guardan en una estructura llamada PCB (cada proceso tiene uno propio). La PCB guarda información como el estado del proceso, el PC, registros, file descriptors abiertos, pid, información de la memoria del proceso.

Luego, existe una tabla de procesos que contiene la información de todos los procesos que están corriendo en el sistema donde cada entrada de la tabla es una PCB. 

![context_switch](/Resumenes/public/context_switch.png)


<h2>Syscalls</h2>
Son herramientas que nos brinda el sistema operativo para poder solicitar acceso a algún recurso privilegiado,
aprovechando las capacidades especiales del kernel. En particular, proveen una interfaz a los servicios que
ofrece el sistema operativo, por ejemplo, realizar una escritura sobre un dispositivo de E/S.

Un proceso puede hacer llamadas al sistema (como fork, exec, write, etc). En todas ellas se debe llamar al kernel y requiere un cambio de privilegio, de contexto y a veces una interrupción.

Las syscalls proveen una interfaz a los servicios brindados por el sistema operativo: la API del SO.
La biblioteca estándar de C incluye funciones que no son syscalls, pero las utilizan para funcionar. Por ejemplo, printf() invoca a la syscall write().

Las system calls disponibles varían de un sistema operativo a otro, aunque la mayoría de conceptos detrás de las system calls suelen ser similares. Buscando que los programas sean portables entre los distintos sistemas operativos, se creó un estándar llamado POSIX. Decimos que un sistema es POSIX compatible si implementa la interfaz de programación de aplicaciones (API) descrita en el estándar POSIX.

Para hacer una syscall el software y hardware hacen lo siguiente:
interrupcion (cambio de contexto), la atrapa el kernel, cambio de privilegio, corre la rutina de atencion (el código de la syscall), vuelve a la función (cambio de contexto y de privilegio)

y en las diapos decia q los parametros se pasan usando registros o una tabla en memoria

<h2>Un poco de E/S y señales</h2>
La E/S es muy lenta. Quedarse bloqueado esperando es un desperdicio de tiempo. Existen otras alternativas:

* Busy waiting: el proceso se queda esperando en un loop infinito hasta que la E/S esté lista.
* Polling: el proceso libera la CPU, pero se fija periódicamente si la E/S está lista. 
* Interrupciones: (permite la multiprogramación). El SO no otorga más quantum al proceso hasta que su E/S esté lista. El HW avisa que la E/S terminó mediante una interrupción y el SO "despierta" al proceso.

Multiprogramación: la capacidad de un SO de tener varios procesos en ejecución. Hay 2 formas de hacer esto desde el código:
* Bloqueante: hago el system call, me bloqueo y cuando termina la E/S el SO me despierta.
* No bloqueante: hago el system call que retorna enseguida y puedo hacer otras cosas. Debo enterarme de alguna manera si mi E/S terminó.

Señales: mecanismo que incorporan los sistemas operativos POSIX, y que permite notificar a un proceso la ocurrencia de un evento. Cuando un proceso recibe una señal, su ejecución se interrumpe y se ejecuta un handler. Cada tipo de señal tiene asociado un handler por defecto, que puede ser modificado mediante la syscall signal(). Toda señal tiene un número asociado que lo identifica. Hay señales que no pueden ser ignoradas ni reemplazadas sus handlers como (SIGKILL, SIGSTOP). Se puede envíar una señal a un proceso mediante la syscall kill().

strace: herramienta que nos permite ver las syscalls que hace un programa. Por ejemplo, strace ls nos muestra las syscalls que hace ls.

ptrace: para monitorear un proceso. Permite monitorear señales, syscalls e instrucciones. Cuando se genera uno de estos eventos el proceso hijo se detiene y el padre tiene que reanudarlo. Tambien se puede modificar el estado del proceso hijo.



<h3>Modificaciones para soportar threads</h3>
 un thread es la unidad básica de utilización de la CPU.
En los sistemas operativos que soportan threads, la PCB debe expandirse para incluir información de cada thread individual. 

Un thread es la unidad básica de utilización de CPU en un sistema. Se compone de un thread id (tid), un program counter, un set de registros, y un stack. 
Otros elementos como el mapeo de memoria (la sección de código, sección de datos, área de heap), y recursos como archivos abiertos, son del proceso que tiene los threads, y son compartidos entre todos los threads de un mismo proceso. 

De modo que, para expandir la PCB y soportar threads, debe haber entradas para cada thread para almacenar (como mínimo): tid, program counter y registros, stack pointer, estado de scheduling y pid (o algún puntero) del proceso que contiene al thread. 


Otra: Se agregan las TCB. Como un thread es un hijo del proceso y comparten memoria, no es necesario tener un PCB por cada thread donde se guardan las paginas de memoria que tiene, los archivos abiertos, etc. Lo que si es necesario es lo minimo indispensable para correr ese thread, que se guarda en la TCB: - Program Counter - Stack Pointer - Thread ID - Estado de los registros - Pointer al PCB del padre - Estado en scheduling (BLOCKED, RUNNING, etc)

![threadsPCB](/Resumenes/public/threadsPCB.png)

<h2>Funciones reentrantes</h3>
El término función reentrante hace referencia a que múltiples instancias de la función pueden estar ejecutándose en simultáneo (sin que nada explote). Es decir, se puede "entrar" de nuevo al código antes de haber terminado de ejecutar una instancia anterior. Es un concepto estrechamente relacionado con la sincronización (en particular la exclusión mutua) y el manejo de recursos compartidos. Un ejemplo donde hace falta que el código sea reentrante es cuando hay llamadas recursivas. Además, muchas partes del sistema operativo tienen que ser reentrantes: un ejemplo típico es el código de los drivers.

Relación con threads: si hay chances de que una misma función vaya a ser  ejecutada por distintos threads, y no hay garantías de que lo vayan a hacer de manera exclusiva, entonces el código sí o sí tiene que ser reentrante.


