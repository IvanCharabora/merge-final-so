[Volver](/README.md)

<h1>Sincronización</h1>
Toda ejecución debería dar un resultado equivalente a alguna ejecución secuencial de los mismos procesos. 

Race condition: Una característica del código donde el resultado depende fuertemente del scheduling. Es cuando la salida o estado de un proceso depende de una secuencia de eventos que se ejecutan de forma arbitraria y trabajan sobre un recurso compartido, lo cual puede producir un error cuando estos eventos dan un resultado que no corresponde a alguna secuencia válida de ejecucion.

Una solución es lograr una exclusión mutua mediante secciones críticas. Una sección crítica es un cacho de código tal que solo hay un proceso a la vez en la sección crítica, todo proceso que esté esperando a entrar en la sección crítica va a entrar, ningun proceso fuera de la sección crítica puede bloquear a otro. 

Se puede implementar una sección crítica mediante:
* Sacar las interrupciones del scheduler dentro de la sección crítica --> rompe la multiprogramación
* Locks booleanos: variables booleanas compartidas que cuando quiero entrar la pongo en 1 --> puede pasar que cuando veo un 0 se me acaba el quantum y cuando me toca devuelta ya se metió otro proceso sin que me dé cuenta

//--Opciones que si funcionan-- 
La solución general consiste en obtener un poco de ayuda del hardware.

* Test And Set (TAS): instrucción que permite testear o establecer atómicamente el valor de una variable entera en 1 y devuelve el valor anterior. Es 1 instrucción en ASM.(se resuelve a nivel HW)

Un uso de ejemplo es el siguiente:
```c++
boolean lock; //Compartida

void main(){
    while(true){
        while(TestAndSet(&lock)); //Espera activa, se queda esperando hasta que devuelva un 0 (si devuelve 1 significa que ya estaba lockeado)
        //sección crítica
        lock = false; //Fin sección crítica
    }
}
```

Consume mucha CPU porque hace busy waiting y perjudica al resto de procesos. Una mejor opción es una especie de sleep pero que se despierte cuando el lock sea 0.

Bajo esta idea, Dijkstra propuso bajo un enfoque productor-consumidor (comparten un buffer de tamaño limitado donde el productor pone elementos y el consumidor los saca, entonces nos gustaria una especie de que el consumidor se duerma si no hay elementos y se despierte cuando haya elementos). Entonces con este problema en mente inventó los semáforos.
Un semáforo es una variable entera con las siguientes características: 
* Se puede inicializar con cualquier valor
* Solo se puede manipular mediante wait y signal
* wait(s): while(s<=0) dormir(); s--;
* signal(s): s++; if(alguien espera por s) despertar a alguno;
* Ambas se implementan de manera tal que ejecuten sin interrupciones (atómicas)

Un tipo de semáforo es el mutex, que es un semáforo binario (solo puede tomar 0 o 1). Se usa para implementar secciones críticas.

Un problema que puede pasar son los deadlocks: El proceso A queda esperando al B y el B espera al A. Decimos que un conjunto de procesos está en estado de deadlock cuando cada proceso del conjunto está esperando por un evento que solo puede ser causado por otro proceso de ese conjunto. Un ejemplo de cuando pasa esto es el siguiente:
```c++
void f(){
    mtx.lock();
    f();
    mtx.unlock();
}
//No llega a hacer los unlock 
```
Otro ejemplo es el siguiente:
```c++
//Proceso A
mtx.lock();
mtx2.lock();
//Proceso B
mtx2.lock();
mtx.lock();
```


En la práctica vimos un patron que se suele llamar barrera donde queremos que pase algo (por ejemplo que corra A1 y B1) antes que otra cosa (A2 y B2):

![barrera](/Resumenes/public/barrera.png)

Otro tipo de barrera es cuando quiero que pasen n cosas antes de que todas esas hagan algo (por ejemplo para que unos estudiantes experimenten en un TP primero todos tenian que haber terminado de implementarlo).
```c++
//variables compartidas
int n, mutex = sem(1), barrera = sem(0);

implementarTp()
mutex.wait();
n++;
mutex.signal();
if(n < N) barrera.wait(); //Otra opción es hacer un if(n == N) barrera.signal(N-1);
barrera.signal();
experimentar();
```

<br>

Otras alternativas para implementar secciones críticas son usando los tipos atómicos
* TAS con objetos: objeto o registro atomico (puede ser una cola, ints, bools, etc) que proveen el getAndSet() y testAndSet(). Como son atomicos las operaciones son indivisibles. Por ejemplo un bool atómico se ve de la siguiente forma:
```c++
private bool reg;

atomic bool get(){return reg;}
atomic void set(bool val){reg = val;}
atomic bool getAndSet(bool val){bool old = reg; reg = val; return old;}
atoic bool testAndSet(){return getAndSet(true);}
```

* Spin locks/TASLock: mutex. lock() no es atómico. Hay busy waiting. Tiene menor overhead que los semáforos. Provee las funciones create, lock y unlock. 
```c++
atomic<bool> reg;
void create(){reg.set(false);}
void lock(){
    while(reg.testAndSet());
}
void unlock(){
    reg.set(false);
}
```

* Local Spinning/TTASLock: probar antes de mandarnos con el TAS(). La diferencia es que el lock se implementa de la siguiente forma:
```c++
void lock(){
    while(true){
        while(reg.get());
        if(!reg.testAndSet()) return;
    }
}
```
Es más eficiente porque el while hace get() en lugar de testAndSet(), lee la memoria cache mientras sea verdadero pero igual es caro. 


* Registros Read-Modify-Write atómicos:
```c++
atomic int getAndInc(){
    int old = reg;
    reg++;
    return old;
}

atomic int getAndAdd(int val){
    int old = reg;
    reg += val;
    return old;
}

atomic T compareAndSwap(T old, T nuevo){
    //old es el valor esperado (el último que observe). Si old != actual, no cambio el valor porque alguién se me metió en el medio.
    if(reg == old){
        reg = nuevo;
        return true;
    }
    return false;
}
```

* Colas atómicas: 
```c++
atomic enqueue(T val){
    mtx.lock();
    cola.push(val);
    mtx.unlock();
}

atomic bool dequeue(T *val){
    mtx.lock();
    if(cola.empty()){
        val = NULL;
        mtx.unlock();
        return false;
    }
    *val = cola.pop();
    mtx.unlock();
    return true;
}
```

Condiciones de Coffman: Si evitamos alguna de las 4 condiciones es probable que no tengamos deadlocks pero no es 100% seguro. Si se cumplen todas entonces hay deadlocks. 
* Exclusión mutua: un recurso no puede estar asignado a más de un proceso.
* Hold and wait: los procesos que ya tienen recursos pueden pedir otro.
* No preemption: No hay mecanismo compulsivo para quitarle los recursos a un proceso.
* Espera circular: Tiene que haber un ciclo de N ≥ 2 procesos, tal que Pi espera un recurso que tiene Pi+1.



Problemas de sincronización:
* Race condition
* Deadlock
* Starvation

Prevención:
* Patrones de diseño
* Reglas de programación
* Prioridades
* Protocolo

Detección:
* Analisis de programas
    * Analisis estático
    * Analisis dinámico
* En tiempo de ejecución
    * Preventivo
    * Recuperación


<h2>Problemas clásicos de sincronización</h2>

**Problema turnos**: queremos que cada proceso imprima su número de proceso en orden.
Podemos usar semáforos para resolverlo:
```c++
semaphore sem[n+1];

proc init(){
    for(int i = 0; i < n+1; i++) sem[i] = 0;
    for(int i = 0; i < n; i++) spawn P(i);
    sem[0].signal();
}

proc P(i){
    sem[i].wait();
    print(i);
    sem[i+1].signal();
}
```

**Razonando en paralelo**: está bien este programa? como razonamos sobre un programa paralelo? La dificultad es que los programas paralelos tienen muchas ejecuciones posibles. 

Una ejecución es $\tau$ = $\tau _ {0}$ --> $\tau _ {1}$ ... donde los $\tau _ {i}$ son los diferentes estados del programa.
No podemos decir que la correctitud depende de si todas las ejecuciones llegan a la postcondición ya que nos interesa ver que pasa si hay programas que no terminan o se traban o abortan o mueren por inanición. Entonces se deben cumpir un conjunto de propiedades que se plantean sobre toda ejecución. 

Hay 2 tipos de propiedades:
* **Safety**: las cosas malas no pasan (como deadlock, pérdida de mensajes, etc). 
* **Liveness**: las cosas buenas pasan (como que todos los procesos terminen, que todos los mensajes lleguen, etc). 

Correctitud: demostrar/argumentar que la combinación de estas propiedads nos lleva a el comportamiento deseado.

Contraejemplo: sucesión de pasos que muestra una ejecución del sistema que viola cierta propiedad. Entonces en safety podemos decir que tiene un contraejemplo finito y en liveness los contrajemplos son infinitos.

A veces además se busca probar **fairness** (a todos los procesos les llega su turno en algún momento y con infinita frecuencia).

En el ejemplo de turnos de arriba podemos argumentar por absurdo que un proceso de mayor índice se ejecuta antes que uno de menor pero eso pasa si alguien se despierta antes, lo cual no puede pasar por cumplir safety.


Otra propiedad es **Wait-freedom** donde todo proceso que intenta acceder a la sección crítica, en algún momento lo hace.


Para formalizar las ideas para demostrar se usa un modelo de proceso.

![modelodeproceso](/Resumenes/public/modelodeproceso.png)

Con esto, podemos formalizar algunas propiedades:
* Fairness: para toda ejecución $\tau$ y todo proceso i, si i puede hacer una transición $l_{i}$ en una cantidad infinita de estados de $\tau$, entonces existe un k tal que $\tau _ {k}$ --$l_{i}$--> $\tau _ {k+1}$. Según entiendo es que si un proceso puede hacer una transición en infinitos estados, entonces en algún momento lo hace.

* Exclusión mutua: para toda ejecución $\tau$ y estado $\tau _ {k}$, no puede haber más de un proceso i tal que $\tau _ {k}$(i) = CRIT. O sea, solo un proceso puede estar en la sección crítica.

* Progreso/Lock-freedom: para toda ejecución $\tau$ y estado $\tau _ {k}$, si en  $\tau _ {k}$ hay un proceso i en TRY y ningún otro proceso en CRIT, entonces existe un j mayor a k tl que en el estado $\tau _ {j}$ hay un proceso en CRIT. O sea, si algún proceso está en TRY entonces en algún momento un proceso va a entrar a la sección crítica.

* Progreso global dependiente/starvation freedom: para toda ejecución $\tau$ si para todo estado $\tau _ {k}$ y proceso i tal que $\tau _ {k}$(i) = CRIT entonces existe un j mayor a k tal que $\tau _ {j}$(i) = REM. O sea, si un proceso está en CRIT, entonces en algún momento sale

* Wait-freedom: si $\tau _ {k}$(i) = TRY entonces existe j mayor a k tal que $\tau _ {j}$(i) = CRIT. O sea, si un proceso está en TRY, entonces en algún momento entra.



<br>

Livelock: Continuamente cambian su estado en respuesta a los cambios de estado de los otros
<br>

Después hay propiedades, como por ejemplo si la sección crítica es para M procesos. Entonces en exclusión mutua, en vez de que solo un proceso esté en CRIT, pueden estar M procesos. Y en wait freedom si hay un proceso en TRY y la cantidad de procesos en CRIT es menor a M entonces en algún momento ese proceso entra a CRIT. 

<h2>Problemas clásicos</h2>

<h3> Readers-Writers </h3>
Descripción: hay una variable compartida donde los escritores necesitan acceso exclusivo pero los lectores pueden leer simultáneamente. Entonces se produce la propiedad SWMR (single writer, multiple readers) donde:

* Si un writer está en CRIT, entonces ningún otro proceso puede estar en CRIT.
* Si un reader está en CRIT, entonces todos los demás procesos que están en CRIT son readers.
// Si lo dejamos asi hay inanición de escritores ya que puede pasar que siempre entren lectores y los escritores no pueden entrar nunca. 
* Todo proceso que está en TRY entra a CRIT en algún momento. 

<h3> Dining Philosophers </h3>
Descripción: hay 5 filósofos que comparten 5 tenedores. Cada filósofo necesita 2 tenedores para comer.

COMPLETAR

<h3> Barberia </h3>
Descripción: hay un barbero que duerme hasta que llega un cliente. Si el barbero está durmiendo, el cliente despierta al barbero. Si el barbero está cortando el pelo, el cliente espera. 

COMPLETAR