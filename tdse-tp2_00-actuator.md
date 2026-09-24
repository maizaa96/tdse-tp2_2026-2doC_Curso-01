El código proporcionado implementa una máquina de estados finitos (FSM) no bloqueante para controlar actuadores (específicamente un LED) utilizando estructuras de datos en C.

A continuación, se desglosa el funcionamiento y la evolución de las variables clave solicitadas:

* **`index`**: Es una variable local utilizada en los bucles `for` para recorrer la lista de actuadores. Comienza en 0 durante `task_actuator_init()` y `task_actuator_update()`. Como la macro `ACTUATOR_DTA_QTY` es 1 (basado en el tamaño de la lista de configuración que solo contiene a `ID_LED_A`), `index` únicamente toma el valor 0 en su ciclo de vida.


* **`task_actuator_dta_list[index].tick`**: Su unidad de medida son los milisegundos (mS), lo cual se infiere de los comentarios de temporización en el código y la función `HAL_GetTick()`. En el flujo normal expuesto en estos archivos, esta variable no se incrementa ni evoluciona; solo se asigna al valor `DEL_LED_MIN` (0) si la máquina de estados entra en el caso de error `default`.


* **`task_actuator_dta_list[index].state`**: Representa el estado actual del actuador en la FSM y pertenece al `enum task_actuator_st_t`. En la función de inicialización, arranca en `ST_LED_IDLE`. A medida que el bucle principal llama a `task_actuator_update()`, alterna hacia `ST_LED_ACTIVE` si se solicita el encendido, y vuelve a `ST_LED_IDLE` cuando se solicita el apagado.


* **`task_actuator_dta_list[index].event`**: Almacena el último evento recibido del tipo `enum task_actuator_ev_t`. Inicializa en `EV_LED_IDLE`. Su evolución depende de llamadas externas que soliciten un cambio de estado en el actuador, momento en el cual cambiará a `EV_LED_ACTIVE` o nuevamente a `EV_LED_IDLE`.


* **`task_actuator_dta_list[index].flag`**: Es un valor booleano que actúa como un semáforo indicando si hay un evento sin procesar. Arranca en `false` al inicializar. Se levanta a `true` únicamente cuando se inyecta un nuevo evento externo y vuelve inmediatamente a `false` una vez que la máquina de estados procesa el evento correspondiente.



### Comportamiento de `task_actuator_statechart(uint32_t index)`

Esta función es el núcleo de la máquina de estados finitos que controla el hardware. Utiliza el `index` para acceder a la configuración física (`task_actuator_cfg_t`) y a los datos dinámicos (`task_actuator_dta_t`) del actuador correspondiente. Su comportamiento mediante un bloque `switch` es el siguiente:

* **`ST_LED_IDLE`**: Si el estado actual es reposo, y se detecta una bandera levantada (`flag == true`) con el evento de encendido (`EV_LED_ACTIVE`), la función baja la bandera (`flag = false`), enciende el LED modificando el pin del microcontrolador y pasa al estado `ST_LED_ACTIVE`.


* **`ST_LED_ACTIVE`**: Si está activo, y hay una bandera levantada con el evento de apagado (`EV_LED_IDLE`), baja la bandera, apaga el LED y devuelve el sistema al estado `ST_LED_IDLE`.


* **`default`**: Actúa como un mecanismo de seguridad; si el sistema entra en un estado no reconocido, reinicia el temporizador, el estado, el evento y la bandera a sus condiciones iniciales de reposo.



```c
// Fragmento ilustrativo de la FSM
if ((true == p_task_actuator_dta->flag) && (EV_LED_ACTIVE == p_task_actuator_dta->event))
{
    p_task_actuator_dta->flag = false; // Se consume el evento
    HAL_GPIO_WritePin(...); // Acción sobre el hardware
    p_task_actuator_dta->state = ST_LED_ACTIVE; // Transición de estado
}

```

### Evolución en la interfaz (`task_actuator_interface.c`)

El archivo de interfaz contiene la función `put_event_task_actuator`, la cual es utilizada por otras partes del sistema para comandar el actuador sin intervenir directamente en su bucle de control.

* **`identifier`**: Pertenece al `enum task_actuator_id_t` (donde `ID_LED_A` es 0) y funciona exactamente igual que el `index`, permitiendo acceder directamente a la posición de memoria del actuador objetivo en la lista global `task_actuator_dta_list`.


* **`task_actuator_dta_list[identifier].event`**: Cuando se invoca la función de interfaz, esta variable abandona su valor previo y adquiere el nuevo evento solicitado como argumento (por ejemplo, `EV_LED_ACTIVE`).


* **`task_actuator_dta_list[identifier].flag`**: Tras actualizar el evento, la interfaz fuerza esta bandera a `true`. Esto avisa de manera asíncrona a la función `task_actuator_statechart` (que se está ejecutando continuamente en el bucle principal) que hay una nueva orden lista para ser procesada.
