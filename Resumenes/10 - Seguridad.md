[Volver](/README.md)

<h1>Seguridad</h1>
Protección: Se trata de los mecanismos para asegurarse de que nadie pueda meter los garfios en los datos del otro. Que usuario puede hacer cada cosa. 

Seguridad:  Se trata de asegurarse que quien dice ser cierto usuario, lo sea. También se trata de impedir la destrucción o adulteración de los datos. La seguridad de la información se entiende como la preservación de la integridad (nadie modifica nada sin permiso), confidencialidad (solo los usuarios con permisos acceden) y disponibilidad de la información (nadie puede hacer inutilizable el sistema).

Las tres A: Autenticación (sos quien decis ser? contraseñas, biometría), Autorización (que podes hacer) y Auditoría (dejo registrado que hiciste).

<h2>Criptografía</h2>
Rama de las matemáticas y de la informática que se ocupa de cifrar/descifrar información utilizando métodos y técnicas que permitan el intercambio de mensajes de manera que sólo puedan ser leídas por las personas a quienes van dirigidos. El criptoanálisis es el estudio de los métodos que se utilizan para quebrar textos cifrados con objeto de recuperar la información original en ausencia de la clave. 

Algoritmos de encriptación simétricos: son aquellos que utilizan la misma clave para encriptar y para desencriptar

Algoritmos de encriptación asimétricos: son aquellos que utilizan dos claves diferentes para encriptar y para desencriptar. La clave pública se utiliza para encriptar y la clave privada para desencriptar.

<h3>Funciones de hash</h3>
Convierte un dato de entrada en un hash. Se suele pedir que dado h sea muy difícil encontrar m tal que h=hash(m) y muy difícil de encontrar un m2 distinto a m1 tal que el hash sea el mismo. Se usan para almacenar contraseñas que no se puedan leer. También sirve para enviar esas contraseñas a través de un sistema. 

Ahora esto se puede vulnerar si se usa un diccionario de contraseñas y se tiene acceso a la función de hash. Pues podes ver si encontras una contraseña que matchee con el hash que interceptaste. 
SALT: Se usa una función de hash doble con un valor de semilla como 2do parametro. O también se puede agregar al final de la clave un valor random. Esto hace más dificil usar la estrategia anterior. Además te salva en el caso de que dos usuarios usen la misma contraseña.

SALT te protege de un ataque de tabla de hash, porque este tiene una tabla con contraseñas comunes y sus hashes, pero al intentar comparar alguno de estos con las contraseñas de la base de datos no le va a servir, porque todas las contraseñas tienen un string agregado (SALT), entonces tienen que descifrar tambien el SALT o probar distintos para ver si le pegan a la correcta combinacion para que el hash final llegue a donde quieren. Para que esto sea efectivo el SALT deberia ser diferente para cada password, si es la misma no tiene mucha gracia

RSA: se toman dos números de muchos digitos y se usan como clave publica y privada. Para encriptar un mensaje, interpreto cada letra como si fuera un número, y hago una cuentita que involucra la clave pública del receptor. Para descifrarlo es necesaria la clave privada, y hacer otra cuentita. Muy resumidamente el algoritmo lo que hace es tomar dos números primos grandes, y calcular el producto de ambos. Luego se elige un número e que sea coprimo con (p-1)(q-1). Luego se calcula el inverso de e en (p-1)(q-1). El inverso de e en (p-1)(q-1) es d. La clave pública es (n,e) y la clave privada es (n,d). Para encriptar un mensaje m, se calcula m^e mod n. Para desencriptar un mensaje c, se calcula c^d mod n. Anda bien porque factorizar la clave pública y descubrir el p y el q es NP.

Existe un ataque llamado replay-attack que las funciones de hash no lo impiden. Se trata de que alguien intercepte un mensaje y lo reenvie. La solución es usar metodos basados en Challenge-Response donde el servidor elige un número al azar, que comunica al cliente, el cliente tiene que encriptar la contraseña utilizando ese número como semilla, el servidor hace lo mismo y se fija si coinciden. Este número va cambiando. 


<br><br>
Monitor de referencias: mecanismo responsable de mediar cuando los sujetos intentan realizar operaciones sobre los objetos en función de una política de acceso.

La forma más sencilla de concebir a la autorización es como una matriz de control de accesos (matriz de sujetos x objetos y en las celdas figuran las acciones permitidas). Se usa el principio de mínimo privilegio. Se suele tener permisos default para cuando se crea un objeto. A este esquema se le llama DAC (Discretionary Access Control). La idea es que los atributos de seguridad se tienen que definir explícitamente. El dueño decide los permisos. Otro esquema es MAC (Mandatory Access Control), se lo utiliza para manejar información altamente sensible, lo que se regula es el flujo de la información. Cada sujeto tiene un grado. Los objetos creados heredan el grado del último sujeto que los modificó. Un sujeto sólo puede acceder a objetos de grado menor o igual que el de el.

DAC en unix son los permisos tipo rwxrw-xr--. setuid y setgid son permisos de acceso que pueden asignarse a archivos o directorios en un sistema operativo basado en Unix. Se utilizan principalmente para permitir a los usuarios del sistema ejecutar binarios con privilegios elevados temporalmente para realizar una tarea específica. Si un archivo tiene activado el bit SETUID se identifica con una “s”. sudo Permite la ejecución de comandos en nombre de otro en forma granular.

SetUid: Es un bit que se encuentra en los binarios que si se encuentra en 1 indica que al correr ese binario, este va a tener los permisos del dueño del mismo. Esto sirve por ejemplo para cambiar las contraseñas en linux, ya que uno como usuario no tiene los permisos suficientes para hacerlo, pero puede correr el programa de cambio de contraseñas que tiene el SETUID prendido y superuser como owner, el cual va a permitir cambiar la contraseña ya que tiene los permisos de superuser al ser ejecutado.

MAC en windows (NFTS) define cuatro niveles de integridad: System, High, Medium, Low. Achivos, carpetas, usuarios, procesos, todos tienen niveles de integridad.
El nivel medio es el nivel por defecto para usuarios estándar y objetos sin etiquetas. El usuario no puede darle a un objeto un nivel de integridad más alto que el suyo.


<h2>Errores de implementación</h2>
En general son debilidades más fáciles de entender y solucionar que los errores de diseño. Error común: Hacer suposiciones sobre el ambiente del programa. ej: el usuario va a ingresar una cantidad acotada de caracteres alfanumericos. La entrada puede venir de variables de ambiente, entradas del programa u otras fuentes.
<h3>Format string</h3>
Ocurre cuando no se sanitiza el input de un usuario. Por ejemplo si hay un system(input) puedo poner que abra un shell de root (./bin/sh). El impacto es el escalamiento de privilegios. La solución es validar el input antes de ejecutarlo. Puede ser mediante allowlist (validar que tenga el formato requerido) o blocklist (sanitizar caracteres peligrosos como ; ” & …)

<h3>Environment variables</h3>
Puedo cambiar el PATH a una carpeta por ejemplo /tmp (export PATH= “/tmp:$PATH”) y luego cuando ejecute ./ping se va a ejecutar el de tmp. Entonces puedo hacer un archivo ./tmp/ping que haga lo que quiera, como abrir una shell en root (como system corre en root abrir una shell abre una shell en root). Ahí lo entendí mejor creo, el problema es que ejecuta system(command) donde command es ping -c pero el ping va a ejecutar el archivo nuevo creado.
También es escalamiento de privilegios y la solución es utilizar el path completo (o sea system(/sbin/ping).

<h3>Buffer overflow</h3>
Consiste en que el usuario escriba más que el size del buffer haciendo que escriba en otros sectores de memoria. Por ejemplo, si tengo un struct char password [100]; bool valid puedo hacer que el password escriba el bool valid. Dicho más formalmente, se ingresa un input de usuario directamente sobre un buffer de tamaño limitado. El impacto es la autenticación indebida en el ejemplo de valid. 

Otra definición: Se ingresa input de usuario directamente sobre el stack, sin limitar su tamaño, permitiendo que haya overflow. Esto permite una ejecución arbitraria de código. El atacante puede pisar la dirección de retorno. Se puede solucionar con scanf que limite la cantidad de caracteres a leer.
//EL STACK VA DE MAYOR ADDR A MENOR. Y  LAS PRIMERAS VARIABLES DECLARADAS SE ALMACENAN EN POSICIONES MAYORES (O SEA LAS VARIABLES DECLARADAS DESPUÉS PUEDEN PISAR A LAS DE ANTES)

Con esto podemos controlar el eip y podemos saltar a nuestro propio buffer (y ejecutar el código que queramos) o saltar a código del programa que haga lo que queremos (como authenticate()).

<h3>SQL injection</h3>
Lo de arriba pero en base de datos. Por ejemplo si hay un select … where … AND pass=”$PASS” entonces si pongo PASS=nose” OR “1”= “1 puedo acceder. Entonces el impacto es que puedo manipular una base de datos: puedo escalar de privilegio, destrucción de datos, denegación de servicio, obtención de datos privados. La solución es sanitizar los datos antes de usarlos.

<h3>Condiciones de carrera</h3>
Se usan las race conditions para vulnerar al sistema.
Ejemplo (típico): Crear el archivo si no existe. La operación debe ser atómica.

Problema: combinado con links, sobreescribir archivos importantes

Explicación ataque: Un proceso intenta crear un archivo. En este caso, el sistema debe comprobar que el proceso tiene los permisos necesarios para escribir en la locación fijada y luego se le permite escribir. El problema es que esto ocurre en momentos distintos y en el intervalo entre que uno ocurre y comienza lo segundo, un atacante podría crear un link símbolico que rediriga el path del nuevo archivo a alguna parte del sistema

<h3>Malware</h3> 
Software diseñado para llevar cabo acciones no deseadas y sin el consentimiento explícito del usuario. Puede venir de páginas webs, adjuntos de emails, vulnerabilidades en software, etc. 



<h2>Mecanismos de protección</h2>

* DEP (Data Execution Prevention): ninguna región de memoria debería ser al mismo tiempo escribible y ejecutable. Se implementan con ayuda del HW. Ya no se puede inyectar código. Hay técnicas para “bypassear” esta protección (como ROP)
* ASLR (Address Space Layout Randomization): modifica de manera aleatoria la dirección base de regiones importantes de memoria entre las diferentes ejecuciones de un proceso (como Heap, Stack, etc). Impide los ataques que utilizan direcciones hardcodeadas (como la de buscar una dirección para modificar el ret o cosas así). Aunque sea más difícil, también puede ser bypasseable
* Stack Canaries: se implementa a nivel compilador. Se coloca un valor en la pila luego de crear el stack frame. Antes de retornar de la función se verifica que el valor sea el correcto. La idea es proteger el valor de retorno de la función de posibles buffer overflows. También es bypasseable.
