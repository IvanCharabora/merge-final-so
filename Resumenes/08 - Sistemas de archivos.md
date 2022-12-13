[Volver](/README.md)

<h1>Sistemas de archivos</h1>

Archivo: secuencia de bytes que se los identifica con un nombre. 

File system: módulo dentro del kernel que se encarga de administrar los archivos en disco. Ejemplos de sistemas de archivos: FAT, NTFS, ext2, ext3, ext4, etc. Una de sus responsabilidades más elementales es ver como se organizan, de manera lógica, los archivos tanto internamente (como se estructura la información dentro del archivo) como externamente (como se ordenan los archivos).

Link: es un alias, otro nombre para el mismo archivo. Teniendo links la estructura deja de ser arbórea y se vuelve un grafo dirigido propiamente dicho, con ciclos y todo.

Además, el FS determina cómo se nombrará a los archivos. Caracteres de separación de directorio, si tienen o no extensión, restricciones a la longitud y caracteres permitidos, distinción o no entre mayúsculas y minúsculas, prefijado o no por el equipo donde se encuentran. 

<h2>Representación de archivos</h2>
Para el FS un archivo es una lista de bloques + metadata. La forma más sencilla es colocar los bloques contiguos en el disco donde las lecturas son insuperablemente rápidas pero tengo fragmentación y el problema de que pasa si el archivo crece. 

El paso obvio es hacer una lista enlazada pero las lecturas aleatorias son lentas y se desperdicia espacio al contener los punteros. Una vuelta de tuerca a esto es tener una tabla donde contiene la información de que bloque le sigue a cada bloque. Este método es usado por FAT. Esto soluciona lo antes dicho pero tengo que tener toda la tabla en memoria, puede ser inmanejable para discos grandes. Además al ser una única tabla hay mucha contención. Y es poco robusto, si el sistema cae, la tabla estaba en memoria. Por último, no maneja seguridad. En FAT se manejan los directorios indicando el primer bloque de cada archivo.

La solución Unix son los inodos y manteniendo indexados todos los bloques de cada archivo en un inodo. Cada archivo tiene un inodo. En las primeras entradas hay atributos (tamaño, permisos, etc).  Luego están las direcciones de algunos bloques (permite acceder rápido a archivos pequeños). Sigue una entrada que apunta a un bloque llamados single indirect block (punteros a bloques de datos, sirve para archivos hasta 16MB), luego una entrada double indirect block (apunta a una tabla de single indirect blocks, cubren archivos hasta 32GB) y luego le sigue triple indirect block (apunta a un bloque de double indirect blocks, cubre 70TB). Los inodos permiten tener en memoria solo las tablas correspondientes a los archivos abiertos.

![inodos](/Resumenes/public/inodos.png)

Directorios: En inodos se reserva un inodo como entrada al root directory. Por cada archivo o directorio dentro del directorio hay una entrada. Dentro del bloque se guarda una lista de inodos.

![dirInodos](/Resumenes/public/dirInodos.png)

![diagGeneralInodos](/Resumenes/public/diagGeneralInodos.png)

El superblock contiene metadatos críticos del sistema de archivos tales como información acerca del tamaño, cantidad de espacio libre y donde se encuentra los datos. 

![inodos2](/Resumenes/public/inodos2.png)

Links en inodos:
* hard links: Dos directorios apuntan al mismo inodo. 
* symbolic links: Tengo un directorio que apunta a traves de un inodo a otro directorio que apunta a su inodo.

![linksInodos](/Resumenes/public/linksInodos.png)



Cuando se habla de la metadata generalmente hablamos de los inodos, permisos, tamaños, propietarios, fecha de creación, tipo de archivo, flags, etc. 

<br>
<h3>Manejo del espacio libre</h3>
Otro problema es cómo manejar el espacio libre. Una técnica posible es utilizar un mapa de bits empaquetado,
donde los bits en 1 significan libre pero requiere tener el vector en memoria. Podemos tener una lista enlazada de bloques libres. Tengo un puntero al primer bloque libre en memoria. Pero si de repente el SO necesita varios bloques ahí no está tan bueno porque tengo que hacer varias consultas al disco. En general se clusteriza. Es decir, si un bloque de disco puede contener n punteros a otros bloques, los primeros n − 1 indican bloques libres y el último es el puntero al siguiente nodo de la lista. Así si necesito varios bloques con una consulta al disco puedo conseguir n. Un refinamiento consiste en que cada nodo de la lista indique, además del puntero, cuantos bloques libres consecutivos hay a partir de él.

<br>
<h3>Caché</h3>
Una manera de mejorar el rendimiento es usando una caché donde tengo una copia en memoria de los bloques del disco. Se maneja muy similar a las páginas. Un efecto muy interesante del cache es que puede grabar las páginas de manera ordenada, de manera tal que el administrador de E/S pueda planificar más eficientemente la escritura.  A veces, las aplicaciones pueden configurarse para hacer escritura sincrónica, es decir, escribiendo en disco inmediatamente. Esto es mucho más lento. Por eso es mejor hacer escritura asincrónica donde las modificaciones se bajan a la cache y después se bajan a disco.

<br>
<h3>Consistencia</h3>
Los datos se pierden y eso puede romper el filesystem. Por eso se provee el system call fsync(), para indicarle al SO que queremos que las cosas se graben si o si. Es decir, que grabe las páginas “sucias” del caché. Sin embargo, el sistema podría interrumpirse en cualquier momento. 

La alternativa más tradicional consiste en proveer un programa que restaura la consistencia del FS. Básicamente, recorre todo el disco y por cada bloque cuenta cúantos inodos le apuntan y cuántas veces aparece referenciado en la lista de bloques libres. Dependiendo de los valores de esos contadores se toman acciones correctivas, cuando se puede. La idea es agregarle al FS un bit que indique apagado normal. Si cuando el sistema levanta ese bit no está prendido, algo sucedió y se debe correr este programa. El problema es que eso toma mucho tiempo y el sistema no puede operar normalmente hasta que este proceso termine.
Hay algunas alternativas para evitar eso, total o parcialmente:
* soft updates: se trata de rastrear las dependencias en los cambios de la metadata para grabar solo cuando hace falta. Sigue haciendo falta una recorrida por la lista de bloques libres, pero se puede hacer mientras el sistema está funcionando.
* journaling: Algunos FS llevan un log o journal. Es decir, un registro de los cambios que habría que hacer. Eso se graba en un buffer circular. Cuando se baja el caché a disco, se actualiza una marca indicando qué cambios ya se reflejaron. Hay un impacto en performance pero es bajo porque este registro escribe en bloques consecutivos. 

<br>
<h3>NFS</h3>
es un protocolo que permite acceder a FS remotos como si fueran locales utilizando RPC. La idea es que un FS remoto se monta en algún punto del sistema local y las aplicaciones acceden a archivos de ahí, sin saber que son remotos. Para poder soportar esto, los SO incorporan una capa llamada Virtual File System. Esta capa tiene vnodes por cada archivo abiertos.  Así, los pedidos de E/S que llegan al VFS son despachados al FS real, o al cliente de NFS, que maneja el protocolo de red necesario. 