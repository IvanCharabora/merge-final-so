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

<h3>Scheduling en Procesos</h3>
Acordarse que solo un proceso a la vez puede estar en la CPU. Entonces hay que dejarlo un "ratito", a ese "ratito" lo llamamos quantum. En general, cuando se acaba el quantum le toca el turno al siguiente proceso. 

Scheduler: componente esencial del kernel. Su función es decidir a que proceso le corresponde ejecutar en cada momento. Hay varias formas de decidir esto. Que proceso elegir depende de la política de scheduling que se esté usando. 

Para cambiar el programa que se ejecuta en la CPU debemos:
* guardar los registros 
* guardar el IP
* si se trata de un programa nuevo, cargar el nuevo programa en memoria
* cargar los registros
* cargar el IP

A esto se le llama context switch. El IP y demás registros se guardan en una estructura llamada PCB (cada proceso tiene uno propio). La PCB guarda información como el estado del proceso, el PC, registros, etc.

Luego, existe una tabla de procesos que contiene la información de todos los procesos que están corriendo en el sistema donde cada entrada de la tabla es una PCB.

![context_switch](/Resumenes/public/context_switch.png)


<h2>Syscalls</h2>
Son herramientas que nos brinda el sistema operativo para poder solicitar acceso a algún recurso privilegiado,
aprovechando las capacidades especiales del kernel. En particular, proveen una interfaz a los servicios que
ofrece el sistema operativo, por ejemplo, realizar una escritura sobre un dispositivo de E/S.

Un proceso puede hacer llamadas al sistema (como fork, exec, write, etc). En todas ellas se debe llamar al kernel y requiere un cambio de privilegio, de contexto y a veces una interrupción.

Las syscalls proveen una interfaz a los servicios brindados por el sistema operativo: la API del SO.
La biblioteca estándar de C incluye funciones que no son syscalls, pero las utilizan para funcionar. Por ejemplo, printf() invoca a la syscall write().


<h2>Un poco de E/S y señales</h2>
La E/S es muy lenta. Quedarse bloqueado esperando es un desperdicio de tiempo. Existen otras alternativas:

* Busy waiting: el proceso se queda esperando en un loop infinito hasta que la E/S esté lista.
* Polling: el proceso libera la CPU, pero se fija periódicamente si la E/S está lista. 
* Interrupciones: (permite la multiprogramación). El SO no otorga más quantum al proceso hasta que su E/S esté lista. El HW avisa que la E/S terminó mediante una interrupción y el SO "despierta" al proceso.

Multiprogramación: la capacidad de un SO de tener varios procesos en ejecución. Hay 2 formas de hacer esto desde el código:
* Bloqueante: hago el system call, me bloqueo y cuando termina la E/S el SO me despierta.
* No bloqueante: hago el system call que retorna enseguida y puedo hacer otras cosas. Debo enterarme de alguna manera si mi E/S terminó.

Señales: mecanismo que incorporan los sistemas operativos POSIX, y que permite notificar a un proceso la ocurrencia de un evento. Cuando un proceso recibe una señal, su ejecución se interrumpe y se ejecuta un handler. Cada tipo de señal tiene asociado un handler por defecto, que puede ser modificado mediante la syscall signal(). Toda señal tiene un número asociado que lo identifica. Hay señales que no pueden ser ignoradas ni reemplazadas sus handlers como (SIGKILL, SIGSTOP). Se puede envíar una señal a un proceso mediante la syscall kill().
