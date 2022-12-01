<h1>Scheduling</h1>
La política de scheduling es una de las principales huellas de identidad en un SO. Se puede optimizar distintos aspectos como:

* Ecuanimidad (fairness): dosis "justa" de CPU para cada proceso
* Eficiencia: que la CPU esté ocupada la mayor cantidad de tiempo posible
* Carga del sistema: minimizar cantidad de procesos listos esperando CPU
* Tiempo de respuesta: minimizar tiempo de respuesta percibiido por el usuario
* Latencia: minimizar tiempo para que un proceso de resultados
* Tiempo de ejecución: minimizar el tiempo total que toma ejecutar completamente un proceso
* Rendimiento: maximizar número de procesos completados por unidad de tiempo	
* Liberación de recursos: liberar recursos tan pronto como sea posible

Muchos de estos objetivos son contradictorios, por lo que es necesario elegir uno o varios de ellos y optimizarlos.

El scheduling puede ser cooperativo o con desalojo.
* Cooperativo: El scheduler analiza la situación cuando el kernel toma control. Las decisiones de scheduling se toman cuando va de un estado running a waiting o cuando termina 
* Con desalojo: En cada interrupción del clock el SO puede elegir si cambiar el proceso actual. El proceso actual puede ser interrumpido por otro proceso que tenga mayor prioridad o por un proceso que haya terminado de ejecutarse. Las decisiones de scheduling se toman cuando va de un estado running a uno ready o cuando va de un estado waiting a uno ready (y las dos mencionadas arriba). La desventaja es que se necesita tener clock y no hay garantías de continuidad de procesos.

Las distintas políticas de scheduling son:
* FIFO/FCFS: atiendo primero al que llega primero hasta que termine o hasta que se bloquee (sin desalojo). Se implementa con una cola de PCBs. Si llega un proceso largo puede hacer de tapón. Se puede resolver con prioridades pero puede producir starvation. Se puede solucionar agregando aging (aumentar la prioridad a medida que envejece).

* Round Robin: parecido al de arriba pero cada proceso tiene un quantum donde una vez transcurrido se desaloja el proceso y se lo agrega al final de la cola lista. Si el quantum es corto el tiempo de context switch es significativo y si el quantum es largo es como FCFS. No produce starvation. Se suele combinar con prioridades que van decreciendo a medida que los procesos reciben su quantum.

* Múltiples colas:  Se tiene varias colas. A cada cola se le asigna un quantum distinto. Tiene prioridad la cola de menor quantum. Los procesos más largos en CPU se pasan a colas con quantum más grande. Lo más prioritario se queda en la cola 1. El manejo de las colas se puede customizar. Los procesos con uso de E/S van a la cola más prioritaria y luego de entran por la cola de mayor prioridad porque se supone que se vuelven más interactivos. 

* Multi level queue: Hay distintas colas con prioridad distinta y en cada una hay una política de scheduling particular. 

* Multi level feedback queue: Los procesos pueden cambiar de cola. Cuando no le alcanza la cpu pasa a la cola siguiente (implementa aging).

* Shortest Job First: Hago primero lo más corto. Si conozco la duración de los procesos de antemano es óptimo. Quiere maximizar el throughput, anda bien en sistemas que usan trabajos batch. Si no conozco la duración de los procesos de antemano es ineficiente. Se puede solucionar con estimación de duración. 

* Shortest Remaining Time First: Es como SJF pero con desalojo.

Casos de scheduling:

* Real Time: son los sistemas donde las tareas tienen una deadlines. Una política posible para esto es Earliest Deadline First.

* Symmetric Multiprocessing: Los procesos se ejecutan en varios procesadores a la vez. Todos los procesadores son iguales y comparten el mismo SO. El problema es que si migras un proceso de CPU perdes la cache. Por eso se utiliza el concepto de afinidad al procesador: tratar
de usar el mismo procesador, aunque se tarde un poco mas en obtenerlo. Puede ser afinidad blanda o dura. Se puede distribuir la carga entre los procesadores con pull/push migration


Scheduling en la práctica:
Elegir un buen algoritmo de scheduling que funcione en la práctica es muy difícil. Suele requerir prueba/error/corrección, y muchas veces deben ajustarse a medida que cambian los patrones de uso. Si bien cada proceso es único, algunas cosas se pueden saber:
* Si un proceso abre la terminal, muy probable esté por convertirse en algo interactivo
* En algunos casos se puede usar análisis estático para ver si cierto comportamiento se va a repetir, si el proceso no tiene pensado terminar, etc.
* Etc.

Linux CFS (Completely Fair Scheduler) Scheduler:
Se tiene un árbol balanceado de busqueda (en realidad un red-black tree, para que se balancee y todas las operacion sean log n) con los procesos listos usando virtual run time como key (vruntime es cuanto tiempo estuvo ejecutando el proceso). La idea es seleccionar el de menor vruntime. Del lado izquierdo del árbol están los procesos con menor vruntime. Esta cacheado el menor de todos para poder accederlo en O(1). Entonces los procesos se van moviendo para la derecha del arbol. Además usa prioridades usando nice values donde el vruntime depende del tiempo que estuvo ejecutando el proceso y el nice value del proceso. 


![cfs](/Resumenes/public/cfs.png)