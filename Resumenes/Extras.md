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


* TRIM: https://hardzone.es/tutoriales/componentes/comando-trim-ssd/ y https://www.techtarget.com/searchstorage/definition/TRIM