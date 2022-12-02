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
