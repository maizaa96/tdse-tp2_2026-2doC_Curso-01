
Imagina que este código es el cerebro de un botón (el sensor) que se comunica con una computadora principal (el sistema) usando un buzón de mensajes (la cola).

**La evolución del botón (`task_sensor_dta_list`)**
Piensa en esto como la memoria personal del botón.

*Al prender el equipo (`task_sensor_init`):*

* **`index`**: Es el número de lista del botón. Como solo hay un botón configurado, es el alumno número 0.


* **`state` (Estado)**: El botón arranca diciendo "estoy en reposo, nadie me está tocando", que en el código es `ST_BTN_IDLE`.


* **`event` (Evento)**: Arranca asumiendo "estoy suelto", es decir, `EV_BTN_UP`.


* **`tick`**: Es un cronómetro interno que mide el tiempo en milisegundos (ms). Al prender el equipo, el código simplemente lo ignora y no le pone un valor inicial.



*Mientras el equipo está funcionando (`task_sensor_update`):*

* **`index`**: Sigue siendo el botón 0.


* **`event`**: El microcontrolador está constantemente mirando el cable físico del botón. Si ve electricidad que indica presión, anota "me apretaron" (`EV_BTN_DOWN`); si no, anota "me soltaron" (`EV_BTN_UP`).


* **`state`**: Cambia dinámicamente entre el reposo (`ST_BTN_IDLE`) y estar activo (`ST_BTN_ACTIVE`).



**El tomador de decisiones (`task_sensor_statechart`)**
Esta es la regla de oro que decide qué hacer paso a paso cuando interactúas con el botón.

* Si el botón estaba tranquilo (`ST_BTN_IDLE`) y de repente nota que tu dedo lo aplasta (`EV_BTN_DOWN`), cambia su estado a "activo" y mete una carta en el buzón del sistema avisando "¡Ey, me acaban de apretar!".


* Si el botón ya estaba apretado (`ST_BTN_ACTIVE`) y nota que quitaste el dedo (`EV_BTN_UP`), cambia su estado de nuevo a tranquilo y mete otra carta en el buzón avisando "Ya me soltaron".


* Si por algún error espacial el botón se confunde de estado, tiene una regla de emergencia (el `default`) que lo reinicia a tranquilo, suelto y pone su cronómetro a 0.



**El buzón de mensajes (`event_task_system_queue`)**
Esta es la fila donde el botón deja sus cartas para que el sistema las lea cuando tenga tiempo. Es un buzón circular donde caben exactamente 16 cartas a la vez.

*Al prender el equipo (`init_event_task_system`):*

* **`head` (La ranura de entrada)**: Arranca en la posición 0. Por aquí entran los mensajes nuevos.


* **`tail` (La puerta de salida)**: Arranca en la posición 0. Por aquí el sistema saca los mensajes para leerlos.


* **`count` (Cantidad)**: Arranca en 0 porque no hay cartas adentro.


* **`queue[i]` (Los 16 casilleros)**: Se rellenan con una etiqueta que dice `EMPTY` (vacío) para indicar que están libres.



*Mientras el equipo está funcionando:*

* Cada vez que el botón te ve apretar o soltar y manda un mensaje, la cantidad de cartas (`count`) suma 1.


* El casillero de entrada actual (`queue[head]`) guarda el mensaje.


* La ranura de entrada (`head`) se mueve un casillero hacia la derecha (suma 1) para dejar espacio al siguiente mensaje. Si llega al fondo (casillero 16), da la vuelta y vuelve a empezar en el casillero 0.


* La puerta de salida (`tail`) no se mueve hasta que otra parte del código venga, saque la carta, la lea y deje el casillero vacío de nuevo.
