[Volver](/README.md)

<h1>Sistema de E/S + drivers</h1>
Hay muchos tipos de dispositivos de E/S que pueden ser categorizados como de almacenamiento, comunicaciones y otros. Nos vamos a concentrar principalmente en los de almacenamiento.Tradicionalmente los SO se tenían que preocupar de discos rígidos, unidades de cinta, discos virtuales, etc. Para nosotros un dispositivo de E/S va a tener el dispositivo físico y un controlador del dispositivo (interactúa con el SO mediante algún de bus o registro)

![adminES](/Resumenes/public/adminES.png)

Drivers: componentes de software que conocen las particularidades del hardware contra el que hablan. Corren con máximo privilegio. Pueden interactuar con los dispositivos mediante polling (chequear constantemente si el dispositivo está listo o si se comunicó, facil pero consume mucha CPU), interrupciones (el dispositivo genera una interrupción, la desventaja es que los cambios de contexto son impredecibles) o DMA (controlador DMA y cuando termina intterumpe a la CPU).

El subsistema de E/S se ocupa de proveerle al programador una API sencilla: open, close, read, write, seek. 

Los dispositivos pueden ser:
* char device: Dispositivos en los cuales se transmite la información byte a byte. Ej: teclado, mouse, serial ports, etc. Debido a su acceso secuencial no soportan acceso aleatorio y no utilizan cache
* block device: Dispositivos en los cuales se transmite la información en bloque. Permiten acceso aleatorio y utilizan cache. Ej: discos, CD-ROM, etc.

El diáogo con estos dispositivos tienen las siguientes características:
* Son de lectura, escritura o lecto-escritura.
* Brindan acceso secuencial o aleatorio: Un dispositivo secuencial transfiere información en un orden determinado por el dispositivo, mientras que los usuarios de un dispositivo de acceso aleatorio pueden pedir que se busque cualquier porción de la memoria disponible.
* Son compartidos o dedicados: Un dispositivo compartido puede ser usado por varios procesos o thread de manera concurrente, mientras que uno dedicado no. 
* Permiten una comunicación de a caracteres o de a bloques
* La comunicación es sincrónica o asincrónica: un dispositivo síncrono realiza las transferencia con un tiempo de respuesta predecible y coordina con otros aspectos del sistema. Un dispositivo asíncrono exhibe tiempos de respuesta irregulares que no se pueden coordinar con otros eventos del sistema.
* Tienen distinta velocidad de respuesta

Una de las funciones del SO, en tanto API de programación, es brindar un acceso consistente a toda la fauna de dispositivos ocultando las particularidades de cada uno de ellos tanto como sea posible.

Una de las claves para obtener un buen rendimiento de E/S es manejar apropiadamente el disco. Queremos minimizar los movimientos de la cabeza. Por eso, la planificación de disco se trata de cómo manejar la cola de pedidos de E/S para lograr el mejor rendimiento posible. Las distintas políticas de planificación son:
* FIFO
* SSTF (shortest seek time first): puede producir inanición
* Scan/ascensor: ir primero en un sentido, atendiendo los pedidos que encuentro en el camino, luego en el otro. El tiempo de espera no es tan uniforme.

En la práctica esto se usa combinado con prioridades y otras hierbas.

Gestión del disco: 
* Formateo: Se trata de poner en cada sector unos códigos que luego sirven a la controladora de disco para efectuar detección y corrección de errores. (para ver si anda mal)
* Booteo: programa en ROM que carga a memoria ciertos sectores del comienzo del disco, y los comienza a ejecutar.
* Bloques dañados: el sistema de archivo anota los bloques inválidos. Hay discos que vienen con sectores extra para reemplazar los defectuosos (con una tabla de remapeo para utilizar otro sector)


Spooling: una forma de manejar a los dispositivos que requieren acceso dedicado en sistemas multiprogramados. Un ejemplo es el de la impresora donde cuando un usuario manda a imprimir no queremos que se bloquee hasta que terminen de imprimir los demás. La idea es poner el trabajo en una cola y designar un proceso que los desencole a medida que el dispositivo se libere. Notar que el kernel no se entera que está haciendo spooling pero el usuario si.


Locking: POSIX garantiza que open es atómico y crea el archivo si no existe o falla si ya existe. Esto nos provee un  mecanismo sencillo, aunque no extremadamente eficiente, de exclusión mutua. Suele ser usado para implementar locks.


Protección de la información: tiene sentido dependiendo el valor de la información que estoy protegiendo (cuanto vale para mi, que pasa si se pierde, que cosas no puedo hacer sin ella).

Copias de seguridad: resguardar todo lo importante en otro lado. Toma tiempo. Copiar todos los datos puede ser muy costoso. Entonces suele hacerse una de estas estrategias:
* Una vez al mes/semana/etc hacer una copia total. Para restaurar solo tomo la del día correspondiente
* Todas las noches hacer una copia incremental (solo los archivos modificados desde la última copia incremental). Para restaurar necesito la última copia total y todas las incrementales hasta la fecha.
* Copia diferencial (solo los archivos modificados desde la última copia total). Para restaurar necesito la última copia total y la última diferencial.

Redundancia: la idea en su forma elemental es hacer la escritura en dos discos. Si se rompe tengo el otro. Un método muy común es el RAID. Hay distintos niveles de RAID

* RAID 0 (stripping): no aporta redundancia. Los bloques de un mismo archivo se distribuyen en dos o más discos

    ![raid0](/Resumenes/public/raid0.png)
* RAID 1 (mirroring): espejado de los discos. Mejora el rendimiento de las lecturas.
    
    ![raid1](/Resumenes/public/raid1.png)

* RAID 0+1: combina los dos anteriores. cada archivo está espejado pero al leerlo leo un bloque de cada disco.

    ![raid0+1](/Resumenes/public/raid0+1.png)

* RAID 2 y 3: tener por cada bloque información adicional que permita determinar si se dañó o no.  Además, cierto tipo de errores se pueden corregir automáticamente, recomputando el bloque dañado a partir de la información redundante.  RAID 2 requiere 3 discos de paridad por cada 4 de datos mientras que RAID 3 requiere solo 1. No se usan en la práctica porque tiene mucho costo de cómputo.

    ![raid2](/Resumenes/public/raid2.png)
    ![raid3](/Resumenes/public/raid3.png)

* RAID 4: es como RAID 3 pero hace stripping a nivel de bloque. El disco dedicado a paridad sigue siendo un cuello de botella para el rendimiento, porque todas las escrituras lo necesitan.

    ![raid4](/Resumenes/public/raid4.png)

* RAID 5: el más usado en la práctica junto 0,1,0+1. No hay disco que solo contenga redundancia, si no que lo distribuye en N+1 discos. Soporta la pérdida de un disco cualquiera.

    ![raid5](/Resumenes/public/raid5.png)

* RAID 6: es como RAID 5 pero agrega un segundo bloque de paridad. El objetivo principal es soportar la rotura de hasta dos discos.

    ![raid6](/Resumenes/public/raid6.png)

Sin embargo, RAID no proyege contra borrar un archivo accidentalmente. Por eso se combina con copias de seguiridad. Si la aplicación corrompe los datos, ningún mecanismo sirve. Si se corrompe la estructura interna de los archivos, RAID tampoco ayuda.

Snapshot: Un snapshot de un disco es una copia del archivo de disco de la máquina virtual (VMDK) en un momento concreto. Conserva el sistema de archivos del disco y la memoria del sistema de nuestra VM, permitiéndonos volver a esa imagen guardada en el caso de que algo vaya mal
