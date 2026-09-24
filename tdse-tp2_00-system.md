
El código fuente implementa un programa dividido en dos partes principales que se comunican entre sí: un "Cerebro" llamado Sistema (`task_system`) que toma decisiones, y un "Músculo" llamado Actuador (`task_actuator`) que ejecuta órdenes, como encender o apagar una luz LED. Funciona utilizando una "máquina de estados" (reglas que indican qué hacer según la situación actual) y una "cola de eventos" (un buzón donde los mensajes esperan su turno para ser leídos).

**Evolución de las variables del Sistema (`task_system_dta_list`)**
Al iniciar el programa con la función `task_system_init()`, se preparan las variables base. Durante el funcionamiento continuo (`task_system_update()`), estas variables cambian según los mensajes que llegan.

* **`index`**: Es solo una variable auxiliar de conteo utilizada al arrancar. Empieza en 0 y se usa para configurar la lista de datos del sistema. Como el sistema solo tiene un modo configurado (`SYSTEM_DTA_QTY` vale 1), el bucle solo se ejecuta para `index = 0` y luego desaparece.


* **`task_system_dta_list[index].tick`**: Representa el tiempo medido en **milisegundos (mS)**. Curiosamente, el código de inicialización no le da un valor de arranque a esta variable de forma explícita. Más adelante, si ocurriera un error y el sistema cayera en un estado desconocido (el caso `default`), se le asigna el valor `DEL_SYS_MIN` (que es 0).


* **`task_system_dta_list[index].state`**: Es el "estado de ánimo" del sistema. Arranca en `ST_SYS_IDLE` (Inactivo/Reposo). Durante los ciclos del programa, si recibe la orden de activarse, cambia a `ST_SYS_ACTIVE` (Activo). Si luego le ordenan descansar, vuelve a `ST_SYS_IDLE`.


* **`task_system_dta_list[index].event`**: Es el último mensaje o evento que el sistema ha leído. Arranca vacío o inactivo como `EV_SYS_IDLE`. Cuando revisa su buzón durante el ciclo normal y encuentra un mensaje nuevo, esta variable se actualiza con dicho mensaje (por ejemplo, `EV_SYS_ACTIVE`).


* **`task_system_dta_list[index].flag`**: Es una simple bandera o alarma que dice "¡Tengo un mensaje nuevo!". Arranca apagada (`false`). Cuando el sistema saca un mensaje del buzón, se enciende (`true`). Apenas el sistema lee el mensaje y actúa en consecuencia, él mismo apaga la bandera (`false`) para no repetir la acción.



**Comportamiento de la función de máquina de estados (`task_system_normal_statechart`)**
(Nota: El código incluye `task_system_normal_statechart` en lugar de `task_system_statechart`, pero su propósito es el mismo).
Esta función es el cerebro en acción tomando decisiones paso a paso:

1. Primero, pregunta: "¿Hay mensajes nuevos en mi buzón?" (`any_event_task_system()`).


2. Si la respuesta es sí, saca el mensaje, lo guarda en su memoria (`event`) y levanta su bandera (`flag = true`) para recordar que tiene que procesarlo.


3. Luego, revisa su estado actual:


* Si está inactivo (`ST_SYS_IDLE`), tiene la bandera levantada y el mensaje dice "¡Actívate!" (`EV_SYS_ACTIVE`), entonces: baja la bandera, le envía una orden al actuador para que encienda el LED (`EV_LED_ACTIVE`) y cambia su estado a Activo.


* Si está activo (`ST_SYS_ACTIVE`), tiene la bandera levantada y el mensaje dice "¡Descansa!" (`EV_SYS_IDLE`), entonces: baja la bandera, le dice al actuador que apague el LED (`EV_LED_IDLE`) y cambia su estado a Inactivo.





**Evolución de la cola de mensajes (`event_task_system_queue`)**
Esta estructura funciona exactamente como la fila de un banco circular con 16 sillas.

* **`i`**: Es una variable que solo se usa al principio (`init_event_task_system()`) para contar del 0 al 15 y poner todas las "sillas" vacías.


* **`event_task_system_queue.queue[i]`**: Son las 16 sillas. Al inicio se llenan con el valor `EMPTY` (255). Durante la ejecución, se van llenando con eventos reales (como 0 o 1) y vaciando (volviendo a 255) a medida que el sistema los lee.


* **`event_task_system_queue.count`**: Es la cantidad de personas en la fila. Arranca en 0. Sube en 1 cada vez que llega un mensaje y baja en 1 cuando el sistema lo atiende.


* **`event_task_system_queue.head` (Cabeza)**: Es el acomodador que dice dónde se debe sentar el próximo mensaje que llegue. Arranca en la silla 0. Avanza un lugar cada vez que llega un mensaje. Si llega a la silla 16, da la vuelta y vuelve a la silla 0.


* **`event_task_system_queue.tail` (Cola)**: Es la cajera que atiende al próximo mensaje. Arranca en la silla 0. Avanza un lugar cada vez que lee un mensaje. Al igual que la cabeza, si llega al 16, vuelve a arrancar desde el 0. Si la cajera (`tail`) alcanza al acomodador (`head`), significa que la fila está vacía.



**Evolución de las variables del Actuador (`task_actuator_dta_list`)**
A diferencia del sistema, el actuador no tiene un buzón de mensajes con fila de espera; es como un soldado que recibe una orden directa y se la anota en un papel.

* **`identifier`**: Representa a qué actuador específico se le está dando la orden. En este código, siempre se utiliza el valor `ID_LED_A` (que internamente vale 0) para referirse al primer LED.


* **`task_actuator_dta_list[identifier].event`**: Es la orden directa que recibe. Cambia cuando el Sistema llama a la función `put_event_task_actuator()`. Puede cambiar a `EV_LED_ACTIVE` (encender) o a `EV_LED_IDLE` (apagar).


* **`task_actuator_dta_list[identifier].flag`**: Es la notificación de que llegó una nueva orden para el actuador. Cada vez que recibe un nuevo evento a través de la función `put_event_task_actuator()`, esta bandera se pone en `true`. (Aunque no se muestra aquí cómo se apaga, otro fragmento de código del actuador la leería y la pondría en `false` después de ejecutar la orden).
