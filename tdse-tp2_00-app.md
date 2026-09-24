Vamos a imaginar nuevamente que el microcontrolador es una fábrica, pero ahora nos enfocaremos en los obreros (las tareas), el supervisor (el código de la aplicación), un cronómetro de ultra precisión, y un periodista que anota lo que pasa.

**¿Qué hace cada archivo?**

* **app.c (El manual operativo del supervisor):** Este archivo organiza el trabajo de la fábrica. Tiene una lista de tareas a ejecutar (leer sensores, sistema, mover actuadores) configurada en `task_cfg_list`. Cuando la fábrica arranca (`app_init`), inicializa todo. Luego, en el ciclo de trabajo (`app_update`), el supervisor verifica si ya pasó tiempo suficiente para mandar a los obreros a trabajar, y usa un cronómetro para medir exactamente cuánto tardan en hacer su tarea.


* **app_it.c (El buzón de avisos):** Aquí llegan los avisos externos. La función más importante es `HAL_SYSTICK_Callback()`, que es llamada automáticamente por el metrónomo de hardware (SysTick) cada vez que pasa un milisegundo de tiempo real.


* **systick.c (El botón de pausa):** Contiene una función `systick_delay_us()` que permite congelar por completo el procesador durante una cantidad específica de microsegundos.


* **dwt.h (El cronómetro atómico):** El "Data Watchpoint and Trace" (DWT) es un hardware especial del procesador que cuenta los "latidos" individuales (ciclos) del cerebro del microcontrolador. Este archivo contiene funciones para encenderlo, reiniciarlo a cero, y leer cuántos microsegundos han pasado, ofreciendo una precisión muchísimo mayor que el SysTick.


* **logger.c y logger.h (El periodista):** Estas herramientas permiten enviar mensajes de texto a la computadora (como `LOGGER_INFO`). Para que el mensaje no salga cortado, el periodista grita "¡Alto todos!", deshabilitando las interrupciones (`__asm("CPSID i")`) mientras escribe, y luego vuelve a habilitar el funcionamiento de la fábrica (`__asm("CPSIE i")`).



---

**La evolución de las variables paso a paso**

1. **g_app_tick_cnt (Unidad: Ticks / Milisegundos):**
* **Inicio:** En `app_it_init()` arranca valiendo cero (`ZERO`).


* **Evolución:** Como funciona en segundo plano, la interrupción del hardware suma `+1` a esta variable cada 1 milisegundo. En el ciclo principal `app_update()`, el código de la aplicación verifica si esta variable es mayor a cero; si lo es, le resta `-1` (para procesar ese "tick") y da la orden de ejecutar las tareas. Es básicamente un sistema de fichas: el hardware pone fichas en la mesa y el software las consume para trabajar.




2. **index (Unidad: Índice / Adimensional):**
* **Inicio y Evolución:** Es simplemente un contador interno que va de 0 a 2 (ya que hay 3 tareas definidas en `TASK_QTY`). Se usa en un bucle `for` para recorrer las tareas de sensores (0), sistema (1) y actuadores (2), tanto al iniciar en `app_init()` como al actualizar en `app_update()`.




3. **task_dta_list[index].NOE (Number of Executions - Unidad: Adimensional):**
* **Inicio:** Se inicializa en 0 (`TASK_X_NOE_INI`) en `app_init()`.


* **Evolución:** Cada vez que el bucle principal manda a ejecutar la tarea correspondiente, se le suma `+1` (`NOE++`). Representa cuántas veces ha trabajado ese obrero.




4. **task_dta_list[index].LET (Last Execution Time - Unidad: Microsegundos / uS):**
* **Inicio:** Se inicializa en 0 (`TASK_X_LET_INI`).


* **Evolución:** Justo antes de ejecutar una tarea, el cronómetro DWT se reinicia a cero. Al terminar la tarea, se lee el tiempo en microsegundos y se guarda aquí (`cycle_counter_get_time_us()`). Siempre muestra cuánto tardó el obrero la última vez que trabajó.




5. **task_dta_list[index].BCET (Best-Case Execution Time - Unidad: Microsegundos / uS):**
* **Inicio:** Se inicializa artificialmente alto, en 1000 microsegundos (`TASK_X_BCET_INI`).


* **Evolución:** Después de calcular el `LET` (tiempo actual), el supervisor compara: si el tiempo actual (`LET`) es menor que el mejor tiempo registrado (`BCET`), entonces actualiza el `BCET` con este nuevo récord de velocidad. Guarda el tiempo más rápido histórico.




6. **task_dta_list[index].WCET (Worst-Case Execution Time - Unidad: Microsegundos / uS):**
* **Inicio:** Se inicializa en 0 (`TASK_X_WCET_INI`).


* **Evolución:** Si el tiempo de la última ejecución (`LET`) resulta ser mayor que el peor tiempo registrado (`WCET`), el récord negativo se actualiza. Registra lo máximo que ha llegado a tardar la tarea.




7. **g_app_runtime_us (Unidad: Microsegundos / uS):**
* **Inicio:** Al comenzar la ronda de trabajo en `app_update()`, se reinicia a 0.


* **Evolución:** A medida que cada tarea termina y genera su `LET`, esos microsegundos se van sumando (`+=`) a `g_app_runtime_us`. Representa el tiempo total que tomó ejecutar todas las tareas juntas en un solo ciclo (la suma del tiempo del sensor + sistema + actuador).





---

**El impacto de usar LOGGER_INFO() en los tiempos**

Si colocas un mensaje de `LOGGER_INFO()` adentro del código de una de tus tareas, el impacto es catastrófico para la medición de tiempos.

Como vimos, `LOGGER_INFO` apaga momentáneamente las interrupciones del microcontrolador y formatea texto, un proceso que es computacionalmente muy lento. Si el cronómetro DWT está contando el tiempo de tu tarea (`LET`), no se detiene mientras el logger envía texto por el cable serial.

Como resultado directo:

* **En task_dta_list[index].WCET:** Tu tarea ya no tardará los pocos microsegundos normales, sino que registrará un pico altísimo de tiempo porque incluirá lo que tardó la impresora de texto en funcionar. Tu métrica del "peor caso" (`WCET`) se disparará, arruinando las mediciones de rendimiento de esa tarea.


* **En g_app_runtime_us:** Al ser la suma total de los tiempos de las tareas, esta variable también se inflará dramáticamente durante ese ciclo de trabajo, mostrando que la fábrica entera se detuvo una cantidad exagerada de microsegundos solo para imprimir un mensaje.


### Opción 1: Estructura Jerárquica

- **`task_dta_list`** (`task_dta_t [3]`)
  - **`task_dta_list[0]`** (`task_dta_t`)
    - `NOE` (`uint32_t`): 83967
    - `LET` (`uint32_t`): 4
    - `BCET` (`uint32_t`): 4
    - `WCET` (`uint32_t`): 7
  - **`task_dta_list[1]`** (`task_dta_t`)
    - `NOE` (`uint32_t`): 83967
    - `LET` (`uint32_t`): 3
    - `BCET` (`uint32_t`): 3
    - `WCET` (`uint32_t`): 5
  - **`task_dta_list[2]`** (`task_dta_t`)
    - `NOE` (`uint32_t`): 83967
    - `LET` (`uint32_t`): 2
    - `BCET` (`uint32_t`): 2
    - `WCET` (`uint32_t`): 4

---

### Opción 2: Formato de Tabla

| Elemento | NOE | LET | BCET | WCET | Tipo de Dato |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `task_dta_list[0]` | 83967 | 4 | 4 | 7 | `uint32_t` |
| `task_dta_list[1]` | 83967 | 3 | 3 | 5 | `uint32_t` |
| `task_dta_list[2]` | 83967 | 2 | 2 | 4 | `uint32_t` |
