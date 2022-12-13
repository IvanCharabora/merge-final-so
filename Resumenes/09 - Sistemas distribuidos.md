[Volver](/README.md)

<h1>Sistemas distribuidos</h1>
<h2>Introducción</h2>
Conjunto de recursos conectados que interactúan.

Fortalezas: paralelismo, replicación, descentralización

Debilidades: dificultad para la sincronización, dificultad para mantener coherencia, no suelen compartir clock, información parcial.

Razones para construir sistemas distribuidos:
* Compartir recursos: Si hay varios sitios conectados entre si, entonces un usuario puede usar los recursos disponibles en alguno de los nodos de la red.
* Aceleración de procesamiento: Si hay un proceso que puede ser particionado en varias subprocesos que se pueden correr de manera concurrente, entonces un sistema distribuido puede usar distintos nodos para correrlos simultáneamente para ahorrar tiempo. 
* Redundancia: Los sistemas distribuidos permiten mantener copias redundantes de sus datos en distintos nodos de la red. De esta manera, si un nodo cae, un usuario puede seguir accediendo a la información que necesita sin ningún problema.

Problemas de los sistemas distribuidos:
* Sincronización de eventos: Como ordenarlos eventos cronológicamente. En general, cada nodo de una red tiene su propio clock y es imposible que todos los clock estén sincronizados a la perfección por lo que se deben implementar mecanismos que permitan decidir si un evento en un nodo A es anterior o posterior a un evento en un nodo B.
* Coherencia de datos: Se debe asegurar que los datos sean consistentes entre los distintos nodos de la red. Si se realizan ciertas acciones sobre el sistema, se debe asegurar que todos los nodos afectados reciban la información necesaria para actuar acorde a los cambios que se producen.
* Información parcial: Los datos del sistema están repartidos en toda la red: Ningún nodo tiene toda la información, por esta razón debe saber a quien a recurrir para conseguir los datos específicos que necesita en cada momento. 

<h3>Sistemas distribuidos de memoria compartida</h3>

* Hardware
    * Uniform Memory Access (UMA): Hay un único controlador de memoria. Este se ocupa de asignar la memoria a cada proceso/nodo. 
    * Non-Uniform Memory Access (NUMA): Cada nodo tiene su memoria. Esto hace que cada nodo pueda acceder a su memoria más rápido. 
    * Híbrida
* Software
    * Estructurada
        * Memoria asociativa: Son memorias que están optimizadas para realizar búsquedas a través de todos los datos 
        * Distributed arrays: Son arrays cuyos datos se almacenan en distintos nodos de una red.
    * No estructurada
        * Memoria virtual global: Asigna un namespace a la memoria distribuida en la red. De esta manera, todos los nodos pueden acceder a la misma sin necesitar saber donde se encuentra ubicado el dato que necesita.
        * Memoria virtual particionada por localidad

<br>
<h3>Sistemas distribuidos sin memoria compartida</h3>
Si no hay memoria compartida hay alternativas. La forma en la que coopera el software, en estos casos, se
conoce como arquitectura de software. 

* Conexión Remota: La idea es que los recursos necesarios para cierta parte del procesamiento estén en otro equipo. Entonces, accedemos a este como si estuvieramos sentados frente al mismo y hacemos el procesamiento de manera remota. Notar que involucra al menos dos equipos, pero solo uno de ellos hace el trabajo. El otro simplemente corre un programa interactivo
* RPC: Se trata de un mecanismo que les permite a los programas hacer procedure calls de manera remota. Involucra una serie de bibliotecas que ocultan del programador los detalles de comunicación y le permiten además enviar los datos de un lugar a otro de la red. Notar que es sincrónico
 
En general: estos métodos es que la cooperación tiene la forma de solicitarle servicios a otros. Estos otros no tienen un rol activo. Este tipo de arquitecturas se suelen llamar cliente/servidor.

Para comunicación asincrónica podemos hacer RPC asincrónico o pasaje de mensajes (send/receive)

Pasaje de mensajes: Este es el mecanismo más general porque no supone que haya nada compartido, excepto un canal de comunicación. Problemas a tener en cuenta:
* Tengo que manejar la codificación/decodificación de los mensajes
* Tengo que dejar de procesar para atender el traspaso de mensajes
* La comunicación es lenta
* El canal puede perder mensajes
* Costo económico por mensaje
* Vamos a ignorar en la materia estos pero los nodos pueden morir y la red se puede partir

<h3>Locks en entornos distribuidos</h3>
En entornos distribuidos no hay TestAndSet atómico, por lo tanto hay distintas alternativas:

* La más elemental consiste en poner el control de los recursos bajo un único nodo que hace de coodinador. Cuando un proceso necesita un recurso se lo pide a este nodo que se encarga de "negociar" con los demás nodos para que le den el recurso. Los problemas de esto es que depende de un solo nodo (único punto de falla), cuello de botella en procesamiento, se requiere consultar al coordinador que puede estar lejos aunque los recursos esten cerca, es lento (porque requiere de mensajes que viajen por la red). 
* Enfoque "canto pri". A esto hay que pensar como determinamos quien canto pri antes. Como sincronizar relojes con mucha precisión es caro y dificil surge la idea de orden parcial entre eventos. único importante era saber si algo había ocurrido antes o después de otra cosa, pero no exactamente cuando. Su propuesta es definir un orden parcial no reflexivo entre los eventos de la siguiente manera:
    * Si dentro de un proceso, A sucede antes que B,  A → B.
    * Si E es el envío de un mensaje y R su recepción, E → R. Aunque E y R sucedan en procesos distintos. 
    * Si A → B y B → C, entonces A → C. Si no vale ni A → B, ni B → A, entonces A y B son concurrentes.

    A esto se lo implementa teniendo un reloj en cada procesador (solo importa que sea creciente), cada mensaje lleva el reloj del emisor, como la recepción siempre es posterior al envío si se recibe un mensaje con un tiempo t, se actualiza el reloj interno a t+1. Esto da un orden parcial (si queremos uno total tenemos que desempatar los eventos concurrentes por ejemplo por pid).


<br>

Acuerdo bizantino: Las distintas divisiones del ejército bizantino rodean una ciudad, desde distintas comarcas. Solo pueden ganar si atacan todos juntos, así que deben coordinar el ataque. Sólo se pueden comunicar mediante mensajeros que corren de un lugar a otro, pero pueden ser interceptados. Versión más complicada: pueden ser sobornados. Cómo sabemos que el mensaje llego al otro lado? 

Si los mensajes pueden perderse no existe ningún algoritmo para resolver consenso.

Si los nodos pueden morir pero termina cuando todo proceso que no falla decide en un número finito de pasos, entonces existe un algoritmo para resolver consenso en O((k+1)n*n) donde k es la cantidad de nodos que fallan y n es la cantidad de nodos en total.

Si los nodos no son confiables se puede resolver si y solo si n > 3k y la conectividad es mayor que 2k.


<h3>Scheduling en sistemas distribuidos</h3>
Dos niveles:

* Local: dar el procesador a un proceso listo
* Global: asignar un proceso a un procesador. Hay que compartir la carga entre los procesadores:
    * Estática: en el momento de la creación del proceso
    * Dinámica: varía durante la ejecución 

Migración: 
* iniciada por el procesador sobrecargado
* iniciada por el procesador libre

Política de scheduling:
* Transferencia: cuando hay que migrar un proceso
* Selección: que proceso hay que migrar
* Ubicacion: a donde enviar el proceso
* Informacion: como se difunde el estado


<h2>Problemas de sincronización en sistemas distribuidos</h2>
Modelo de fallas: Cuando se trabaja con algoritmos distribuidos es importante determinar el modelo de fallas. Por ejemplo puede pasar que nadie falla, los procesos se caen y no levantan, se caen y se levantan, se particiona, etc.

Metricas de complejidad: una métrica que suele tener mucho sentido es la cantidad de mensajes que se envían a través de la red. Otra métrica: que tipos de fallas soportan. Otra forma de evaluarlos: cuanta información necesitan.

<h3>Exclusion mutua distribuida</h3>
Hay que mantener la consistencia. 

* más simple: Determino un lider(proxy). Explicado arriba
* Token passing: La idea es armar un anillo lógico entre los procesos y poner a circular un token. Cuando quiero entrar a la sección crítica espero a que me llegue el token. Notar: si no hay fallas, no hay inanición. Desventaja: hay mensajes circulando aun cuando no son necesarios. 
* Solicitud: Cuando quiero entrar a la CRIT envío a todos (incluyendome) solicitud(Pi, ts), siendo ts el timestamp. Cada proceso puede responder inmediatamente o encolar la respuesta. Puedo entrar cuando recibí todas las respuestas. Si entro, al salir, respondo a todos los pedidos demorados. Respondo inmediatamente si: No me interesa entrar en la CRIT o quiero entrar, aun no lo hice y el ts del pedido que recibo es menor que el mío, porque el otro tiene prioridad.  Este algoritmo exige que todos conozcan la existencia de todos
* Locks distribuidos:  Protocolo de mayor, debemos pedir el lock a por lo menos n/2 +1 sitios. Al escribir la copia del objeto, tomamos la versión más alta y la incrementamos en uno. No se puede otorgar dos locks (porque ambos necesitan más de la mitad). No se puede leer una copia desactualizada (significaría que el último que escribió, escribió en menos de n/2 + 1 copias porque si no una de las n/2 + 1 tendría el dato actualizado)

<h3>Elección de líder</h3>
Una serie de procesos debe elegir a uno como líder para algún tipo de tarea. 

* LCR: Cada proceso envía su identificación y número a través del anillo. Cuando un proceso recibe una identificación y un número, compara el número con el suyo. Si el número que llego es mayor al suyo lo reenvía, si es menor envía su identificación y su número (o sea siempre reenvía el mayor). Una vez que le llega su misma identificación y número a un nodo, este se declara líder. El nodo líder envía un mensaje declarándose líder a través del anillo. Cada nodo no líder recibe el mensaje de quien es el líder y lo reenvía. Solo funciona en anillos, O(nn) mensajes.
 
* Flood max: Necesitamos que siempre exista un camino entre dos nodos distintos. Necesitamos saber la longitud del camino más largo (llamemoslo n). Cada nodo manda el máximo valor que vio a todos los nodos con los que está conectado (en primera ronda envía su id). Va actualizando al máximo con los resultados que llegan. Luego de n rondas el algoritmo termina y el máximo id que se propagó por la red es el líder. O(m*n) mensajes con m la cantidad de ejes

* Bully algorithm: Necesitamos que todos los nodos esten conectados con todos los otros. Un nodo va a empezar la elección mandando un mensaje a todos los otros nodos con id mayor que quiere ser líder. Si nadie contesta, es líder. Cada nodo al que le llega el mensaje repite el mismo esquema pero contestandole al nodo que le hablo que va a ser líder. Si nadie contesta se declara líder y broadcastea un mensaje a toda la red declarándose líder. Una elección se puede dar cuenta que el coordinador falló por el timeout. Si el proceso que detectó la falla tiene el identificador más alto de los que quedan manda un mensaje declarándose coordinador a todos. Si uno menor lo hace, anuncia la necesidad de unas elecciones, si no le llega un mensaje donde alguien se declara coordinador, asume que todos los procesos con id mayor fallaron y se declara coordinador. Si llega una respuesta, setea un nuevo timeout para esperar un mensaje del coordinador.

<br>
<br>
Instantanea global consistente (snapshot): Supongamos que tengo un estado E = ΣEi siendo Ei la parte del estado que le corresponde a Pi . Quiero tomar una instantánea global consistente de E. Un proceso se envía a sí mismo un mensaje de marca. Cuando Pi recibe un mensaje de marca por primera vez guarda una copia Ci de Ei y envía un mensaje de marca a todos los otros procesos. En ese momento, Pi empieza a registrar todos los mensajes que recibe de cada vecino Pj hasta que recibe marca de todos ellos. Se puede usar para detección de propiedades estables, detección de terminación, debugging distribuido, detección de deadlocks.

<h3>Protocolos de consenso</h3>
El problema del consenso consiste en poner de acuerdo a múltiples procesos en algo. Es el problema de averiguar cómo un conjunto de procesos de computación aislados que solo pueden comunicarse con mensajes se ponen de acuerdo sobre algo.

* 2PC

    La idea es realizar una transacción de manera atómica. Todos debemos estar de acuerdo en que se hizo o no se hizo.
La idea es, en una primera fase, se le pregunta a todos si están de acuerdo en que se haga la transacción. Caso negativo, se aborta. Si no, se anota quiénes dijeron que sí. Si pasado un tiempo no se reciben todos los sí, también se aborta. Si se recibe sí de parte de todos, se pasa a la segunda fase: se avisa a todos que quedó confirmada.

    ![2pc1](/Resumenes/public/2pc1.png)

    ![2pc2](/Resumenes/public/2pc2.png)

    Si el líder falla luego de la primera fase, los nodos deberán esperar (bloqueando los recursos tomados) hasta que éste se recupere. 2PC no puede recuperarse si fallan el líder y otro nodo más, durante la fase de commit. Si solo fallara el líder, y nadie recibe un mensaje, puede inferirse sin riesgo que no ocurrió ningún commit. En cambio, si fallan el líder y otro más, es posible que el nodo que falló fuera el primero en ser notificado, y hubiera hecho el commit. Aunque se elija un nuevo líder, no puede seguir con la operación de manera segura hasta no obtener confirmación de parte de todos los demás nodos, y por ende debe bloquearse hasta que todos los otros respondan.
    
    2PC resuelve COMMIT con terminación debil (Si no hay fallas, todo proceso decide) pero no satisface que todo proceso que no falla decide (commit con terminación fuerte). La solución 3-PC

* 3PC

    ![3pc](/Resumenes/public/3pc.png)

    ![3pc2](/Resumenes/public/3pc2.png)

![2pc3pc](/Resumenes/public/2pc3pc.png)


<h2>FS distribuidos</h2>
Un sistema de archivos distribuidos es un sistema que provee servicios de archivos (abrir, leer, escribir, cerrar, etc) a sus clientes. Con la particularidad, de que los archivos están repartidos entre todas las máquinas de una red. La interfaz del cliente no debería distinguir entre archivos locales y remotos. Es deber del DFS localizar el archivo y gestionar el transporte de datos.

Modelo cliente-servidor: el servidor almacena tanto los archivos como su metadata en almacenamiento conectado al servidor. Los clientes contactan al servidor para pedirle archivos. El servidor es responsable de la autenticación, el chequeo de permisos, y el envío del archivo. Los cambios que el cliente hace en archivos deben ser propagados al servidor. Este diseño tiene un único punto de falla si el servidor se cae y es un cuello de botella.

Modelo basado en cluster: diseñado para ser más resistentes a fallas y escalable que el modelo cliente-servidor. Los clientes se conectan al servidor de metadata y hay varios servidores de datos que contienen chunks (porciones) de archivos. El servidor de metadata mantiene un mapeo de que servidores
de datos tienen chunks de cada archivo. Los chunks de cada archivo se replican n veces. Un ejemplo de esto es GFS. 


Los desafíos de FS distribuidos incluyen:
* nomenclatura y transparencia.
    
    nomenclatura: mapeo entre objetos lógicos y físicos. 

    mapeo multinivel: abstacción de un archivo que oculta los detalles de como y donde en el disco esta almacenado el archivo.

    Un DFS transparente oculta la ubicación en la que se almacena un archivo en la red.

    Transparencia de la ubicación: El nombre de un archivo no da ninguna pista de cual es la ubicación física en la que está almacenado.

    Independencia de ubicación: No es necesario cambiar el nombre del archivo cuando este es movido entre distintos dispositivos físicos.

    3 enfoques para nombrar los archivos:
    * El más simple, es identificar a los archivos con alguna combinación del nombre de su host y su nombre local, lo que garantiza un nombre único en todo el sistema distribuido. Sin embargo, este esquema no transparente ni independiente de la ubicación física del archivo.
    * Montar los directorios remotos en directorios locales, dando la apariencia de un árbol de directorios coherente. Este método permite la integración transparente de archivos, sin embargo cada máquina debe montar cada directorio remoto a su árbol, lo que puede resultar en diferencias entre los directorios de cada una.
    * Declarar una estructura única global de nombres que abarque todos los archivos del sistema. Sin embargo, este método es difícil de implementar debido a los archivos con nombres especiales y si algún servidor se cae, todos los archivos con nombres almacenados en ese servidor dejarán de ser accesibles.

* acceso a archivos remotos

    Un usuario requiere acceder a un archivo remoto. El servidor que aloja el archivo fue ubicado por el esquema de nombres, y se debe hacer la transferencia de datos. Esto se hace mediante un mecanismo de servicio remoto: los pedidos de acceso son enviados al servidor, el servidor realiza el acceso y el resultado es devuelvo al usuario. El mecanismo más comun de implementar esto es con el paradigma RPC. Para asegurar un rendimiento adecuado del mecanismo, reducir operaciones de disco y disminuir el tráfico dentro de la red, se utilizan cachés en las máquinas de usuario que mantienen una copia local de los accesos realizados recientemente. De esta forma, si la información está cacheada (y es válida), se usa directemente sin hacer ningún pedido al server. Cuando la copia cacheada sufre alguna modificación, ésta se debe ver reflejada en el servidor para mantener consistencia.



* caché y consistencia
    Una de las políticas de actualización más seguras es la write-through que envia las modificaciones al servidor apenas se realizan. Sin embargo, requiere que cada escritura deba esperar a que la información sea recibida por el server por lo que tiene bajo rendimiento. Una alternativa es la política delayed-write o write-back, en la que las modificaciones son escritas más tarde en el servidor. Sin embargo, es más difícil mantener el estado del archivo consistente. Las variantes de esta política pueden ser que se recorra el caché a intervalos regulares y llevar al servidor bloques que se modificaron o otra es write-on-cose que escribe datos en el servidor cuando el archivo es cerrado.

    Al momento de utilizar un archivo, el host debe decidir si la copia de un archivo en su caché es consistente con el archivo guardado en el servidor. Si no lo es, entonces debe actualizar su copia antes de accederlo. Hay dos formas de verificar la validez de la información cacheada:
    * Validación iniciada por el cliente: El cliente contacta al servidor, quien compara las dos copias y responde con una respuesta. La frecuencia de la validación puede ser una vez antes de cada acceso o solo una única vez en el primer acceso. Hay que tener en cuenta que cada verificación demora el acceso y hacerlos de manera muy frecuente puede conllevar a una sobrecarga de la red.
    * Validación iniciada por el servidor: El servidor mantiene un registro de los archivos cacheados en cada cliente. Cuando detecta que puede llegar a haber una inconsistencia en algún archivo, notifica al cliente afectado y deshabilita el cacheo para ese archivo. 

