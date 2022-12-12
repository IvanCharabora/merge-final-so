<h1>Virtualización, contenedores y cloud</h1>
Virtualización: es la posibilidad de que un conjunto de recursos físicos se vean como varias copias de recursos lógicos.
Desde hace rato (por lo menos 1960) tener máquinas virtuales. Ie, de mentira. Los objetivos son variados: 

* Portabilidad
* Simulación
* Aislamiento
* Particionamiento de HW
* Agrupamiento de funciones
* Protección ante fallas de HW
* Migración entre HW sin pérdida de servicio

![vm](/Resumenes/public/vm.png)

Una forma posible de lograr esto es mediante la simulación: en el sistema anfitrión se construye una variable de estado artificial que representa al sistema huésped. Se lee cada instrucción y se modifica el estado como si esta se ejecutase realmente. Sin embargo: El mecanismo puede ser muy lento. ¿Cómo se simulan las interrupciones, DMA, concurrencia, etc.?

Otra forma es mediante la emulación de HW: el sistema emulado se ejecuta realmente en la CPU del anfitrión. Se emulan componentes de HW. Cuando la máquina virtual cree que está haciendo E/S de un dispositivo, en realidad lo está haciendo contra el controlador de máquina virtuales. Problemas: ¿Cómo logro separación de privilegios? Toda la máquina virtual corre en modo usuario. ¿Qué pasa con la velocidad de acceso a los dispositivos?

En este contexto, nace la intención de lograr virtualización asistida por HW, especialmente para lograr evitar los siguientes problemas:
* Ring Aliasing: tengo programas escritos para modo kernel, pero en realidad se están ejecutando en modo usuario. Puedo tener problemas de permisos para ejecutar ciertas instrucciones privilegiadas. 

* Address-space compression: ¿cómo hago para que la máquina virtual no pueda pisar memoria del propio emulador? Recordemos que desde el punto de vista del anfitrión son un único proceso.

* Non-faulting access to privileged state:  algunas instrucciones privilegiadas generan un trap cuando se ejecutan sin permiso. Eso es bueno porque puedo atrapar el trap y simularlas. Pero otras no. ¿Como hago?

* Interrupt virtualization: hay que simularle las interrupciones externas al SO huésped.

* Access to hidden state: hay parte del estado del procesador que no es consultable por software.

* Ring compression: como tanto el kernel huésped como sus programas corren en realidad en el mismo nivel de privilegio, no hay protección entre kernel y programas de usuario. 

* Frequent access to privileged resources: si bien el controlador de máquinas virtuales puede bloquear el acceso a ciertos recursos, haciendo que se genere un trap, esto puede ser un cuello de botella para recursos accedidos frecuentemente.

Para solucionar estos problemas los fabricantes agregaron soporte para la virtualización en el HW. Se agrega Virtual Machine Control Structure (VMCS) en la memoria del procesador. 



<br>
Contenedores: Nos crear un enviroment adecuado para correr alguna aplicación específica. Encapsular un único paquete ejecutables con el el código de la aplicación, los archivos de configuración, librerías y dependencias necesarias para que pueda correr.  Esto es clave si estamos desarrollando en un equipo que tiene un ambiente distinto al que usará nuestra app. Son una forma de virtualizar un SO.

Kubernetes: Plataforma de código abierto para automatizar la implementación, el escalado y la administración de aplicaciones en contenedores. 

Openshift: usa Kubernetes de base, pero agrega restricciones de seguridad por defecto, interface web más completa, manejo de roles, facilidades para el desarrollador

Cloud: Es como una extensión de virtualización porque usa ese concepto como base para su funcionamiento. Brinda storage(public and private), servicios de procesamiento, servicios de aplicaciones, software stack(database server) e infraestructura (server o storage para backups).
