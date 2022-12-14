<h1>Procesos</h1>

* Que es y para que sirve un system call? Explicar los pasos involucrados por hardware y software.

    Un system call es una herramienta que nos brinda el sistma operativo para poder solicitar acceso a un recurso privilegiado aprovechando las capacidades especiales del kernel. O sea, permiten pedirle al kernel distintos servicios, como acceder a disco, crear procesos, etc. La idea es que cuando se usa una syscall se produce una excepción, hay un cambio de privilegio de user mode a kernel mode y el control pasa a ser de un handler provisto por el kernel. 

    Entonces, el software genera una interrupción (trap), transfiriendo el control del usuario al kernel. Se produce un cambio de contexto (como en cualquier interrupción, se guarda el estado actual del procesador para poder retornar sin problemas).

    Un ejemplo de syscall es write(). Por ejemplo, la función printf() de la biblioteca estándar de C invoca a la syscall write(), la cual escribe una cierta cantidad de bytes de un buffer a un archivo referido por un file descriptor pasado por parámetro.
 
* Describa PCB

    El PCB es una estructura de datos que contiene la información necesaria para administrar un proceso. Guarda información como el estado del proceso, el PC, registros, file descriptors abiertos, pid, información de la memoria del proceso.

* Como debe modificarse una PCB para soportar threads?

    Para manejar threads hay que incluir en la PCB del proceso que tiene varios threads una tabla de threads ids de los mismos (cada thread tiene un thread id, un PC, un set de registros y un stack, y comparte con otros threads que pertenecen al mismo proceso la sección de código, la de datos y otros recursos del SO, como archivos abiertos y señales).

* Que son las funciones reentrantes y cual es su relacion con los threads?

    Una función reentrante hace referencia a que múltiples instancias de la función pueden estar ejecutandose en simultáneo. Es decir que se puede "entrar" al código de la función sin que haya terminado la ejecución anterior. Este concepto se puede ver en exclusión mutua o en llamadas recursivas. 

    Relación con threads: si hay chances de que una función pueda ser ejecutada por varios threads y no se asegura que el acceso sea mediante exclusion mutua, entonces la función debe ser reentrante.

* Como modificar algún valor de la PCB?

    COMPLETAR


<h1>Sincronización</h1>   