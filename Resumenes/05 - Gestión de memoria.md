[Volver](/README.md)

<h1>Gestión de memoria</h1>
Como el procesador y la información, la memoria también se comparte tanto para comunicar procesos como para implementar multiprogramación. Para esto, hay que hablar del manejador de memoria o memory manager que se encarga de: 

* Manejar el espacio libre/ocupado
* Asignar y liberar memoria
* Controlar el swapping 

Cuando un proceso está bloqueado y queremos poner a ejecutar otro, que hacemos con la memoria? El mecanismo mas sencillo es el swapping, que consiste en pasar a disco el espacio de memoria de los procesos que no se están ejecutando. El problema de esto es que es muy lento ya que requiere grabar un proceso y luego leer otro aunque tal vez los dos programas entren en memoria. Entonces, en lo posible queremos dejarlos en memoria mientras podamos.

Algunos problemas que aparecen son:
* Reubicación (cambio de contexto, swapping): cuando descargo y cargo una página del disco probablemente quede en otra ubicación. Hay que corregir todas las direcciones.
* Protección (memoria privada de los procesos): que un proceso no lea los datos del otro.
*  Manejo del espacio libre (evitando la fragmentación). La fragmentación es cuando tenemos espacios de memoria libre pero son demasiados chicos como para traer una página nueva. Es decir tenemos mucha memoria libre pero no es continua. Reorganizar la memoria es MUY costoso. La fragmentación puede ser:
    * Interna: espacio desperdiciado dentro de los propios bloques
    * Externa:  bloques libres pero pequeños y dispersos entre los bloques asignados a los procesos
    
    Una posible forma de organizar la memoria para evitar la fragmentación es:

    ![organizacionMemoria](/Resumenes/public/organizacionMemoria.png)

    
Organizar la memoria libre:
* Bitmap: Divido toda la memoria en bloques de 4KB(unidades de reserva). Cada posición del bitmap representa un bloque y me indica si está ocupado o libre. Si bien asignar y liberar es sencillo, encontrar bloques consecutivos no. No es muy usado
* Lista enlazada: Cada nodo de la lista representa a un proceso o a un bloque libre, en cuyo caso figuran el tamaño del bloque y sus límites. Liberar sigue siendo sencillo, asignar es similar una vez que decidí donde. Decidir donde es lo costoso. Esto puede ser con first fit, best fit, worst fit, quick fit (mantengo una lista de los bloques liles de los tamaños mas frecuentes), buddy system (usa splitting de bloques). Todos estos esquemas fallan porque son muy ingenuos, uns producen fragmentación interna y otros externa. En la práctica s eusan esquemas que conocen un poco más sobre la distribución de los pedidos.


<br>
Memoria virtual: Hacerle creer al proceso que dispone de más memoria de la que realmente tiene en cada momento. Antes mencionamos el problema de la reubicación y resuelve el problema de correr un programa que no entra en memoria. La solución consiste en combinar swapping con virtualización del espacio de direcciones. A esto se le lama memoria virtual. Para esto se requiere una unidad llamada Memory Management Unit que es quien se encarga de la traducción de direcciones. 

![mmu](/Resumenes/public/mmu.png)

Cosas:
* Espacio de direcciones: sin memoria virtual es el tamaño de la memoria física. Con memoria virtual es el tamaño de la memoria física + swap y los programas usan direcciones virtuales.
* Obtener una celda: En memoria virtual es poner la dirección en el bus de memora y obtener el contenido. Con memoria virtual es:
    * Poner la dirección en el bus de memoria
    * La MMU traduce la dirección virtual a física
    * La tabla de traducción tiene un bit que indica si el pedazo de memoria está cargado o no. Si no lo está hay que cargarlo
    * La dirección física se pone en el bus de memoria y se obtiene el contenido

    A esto hay un par de detalles:
    * El "pedazo de memoria" en realidad hace referencia a que el espacio de memoria virtual está dividido en bloques de tamaño fijo llamados páginas y el de memoria física en bloques del mismo tamaño llamados page frames. Entonces la MMU traduce páginas a frames. Cuando una página no está en memoria la MMU emite una page fault que es atrapada por el SO. Es el SO el encargado de sacar alguna página de memoria y subir la correspondiente del disco.


La MMU es básicamente su tabla de páginas, que es lo que se usa para el mapeo. Queremos que la búsqueda sea rápida y que la tabla no ocupe mucho espacio. Una solución es tener una tabla de páginas multinivel. Los primeros bits nos llevan hacia la tabla que tenemos que consultar, y ahí usamos el resto de los bits igual que antes. La ventaja es que no hace falta tener toda la tabla en memoria. Se pueden swappear sus partes.

Entradas de las tablas de páginas:
* Page frame (el objetivo de la tabla)
* Bit de presencia
* Bits de protección
* Bit dirty que indica si fue modificada desde que se cargó del disco
* Bit de referenciada que indica si fue usada desde que se cargó del disco

Se usa una caché, llamada TLB para que el acceso a las tablas sea más rápido. Mapea páginas a frames sin consultar tablas.

Para reemplazar las páginas podemos hacer FIFO, segunda oportunidad (FIFO pero si tiene References prendido le doy una segunda oportunidad y la considero como recien subida), Not Recently Used (primero desalojo las que no fueron referenciadas ni modificadas, después las referenciadas y después las modificadas), Least recently used (la que hace más tiempo no se usa)

Reemplazo de páginas: A veces sirve cargar páginas por adelantado en vez de esperar el page fault. Esto en general se hace usando la localidad de referencia (accedemos a páginas contiguas).

En detalle lo que pasa en un page fault:
* Se emite el page fault, que es una interrupción. Lo atrapa el kernel.
* Se salva el estado de la CPU en el stack
* El kernel determina que la interrupción es de tipo page fault, y llama a la rutina específica.
* Hay que averiguar qué dirección virtual se estaba buscando. Usualmente queda en algún registro. Se chequea que sea una dirección válida y que el proceso que la pide tenga permisos para accederla. Si no es así, se mata al proceso. Se selecciona un page frame libre si lo hubiese y si no se libera mediante el algoritmo de reemplazo de páginas.
* Si la página tenía el bit dirty prendido, hay que bajarla a disco. El “proceso” del kernel que maneja E/S debe ser suspendido, generando un cambio de contexto y permitiendo que otros ejecuten. La página se marca como busy para evitar que se use. Cuando el SO es notificado de que se terminó de bajar la página a disco comienza otra operación de E/S, esta vez para cargar la pagina que hay que subir. De nuevo se deja ejecutar a otros procesos. Cuando llega la interrupción que indica que la E/S para subir la página término, hay que actualizar la tabla de páginas para indicar que esté cargada.  
* Cuando le toca al proceso, la instrucción que causó el page fault se recomienza, tomando el IP que había quedado en el stack y los valores anteriores de los registros
* Se devuelve el control al proceso de usuario. En algún momento el scheduler lo va a hacer correr de nuevo, y cuando reintente la instrucción la pagina ya va a estar en memoria.

Thrashing: cuando no alcanza la memoria y hay mucha competencia entre los procesos por usarla y el SO se la pasa cambiando páginas de memoria a disco ida y vuelta(page faults). 

Para  aprotección hay una solución fácil: cada proceso tiene su propia tabla de páginas. No hay forma de acceder a una página de otro. Una manera de solucionarlo es hacer que cada proceso tenga su propio espacio de memoria. Cada uno estos espacios se llama segmentos. Se usa un registro especial para saber a qué segmento hacen referencia las direcciones. Esto no solo facilita la protección. Además permite que cada segmento crezca sin tener que cambiar el programa.

Segmentación: Separo la memoria en segmentos de longitud variable. Son las direcciones lógicas. Sin embargo, tenemos los mismos problemas de fragmentación y swapping que mencionamos antes. Por eso, la alternativa más común es combinar segmentación con paginado. En intel cada proceso tiene su LDT, suele haber un segmento para código, uno para datos y otro para el stack, hay una GDT compartida, cada entrada de la LDT o GDT tiene la base de direcciones y el tamaño del segmento y se usa como física o virtual si está habilitado el paginado.

Segmentación vs paginación:
* Las páginas son invisibles para el programador (assembler), los segmentos no. 
* Los segmentos proveen varios espacios de direccionamiento, que pueden estar solapados. 
* Los segmentos facilitan la protección, aunque también se puede implementar mediante paginado. 
* La segmentación brinda espacios de memoria separados al mismo proceso.


Copy on write: al forkear un proceso se usan las mismas páginas hasta que uno de los dos escribe en una de ellas y entonces se duplican y cada uno queda con su copia. Funciona poniendo las páginas como read only y tan pronto como algunos de los dos necesita realizar una modificación, se genera un protection fault que es interceptado por el sistema operativo.