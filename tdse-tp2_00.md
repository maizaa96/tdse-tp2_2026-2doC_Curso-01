Conversación con Gemini
¿Puedes ayudarme a realizar un Trabajo Práctico sobre Codificación en C de Diagramas de Estado?

¡Por supuesto! Programar un diagrama de estados (o máquina de estados finitos) en C es un excelente ejercicio de lógica y estructura de código.

Para poder guiarte de la mejor manera y darte ejemplos de código que realmente te sirvan para tu trabajo práctico, necesito hacerte un par de preguntas rápidas sobre lo que te pidieron:

Q: ¿Te pidieron usar alguna estructura de programación en particular?
A: (no selection)

Q: ¿En qué etapa estás del trabajo?
A: (no selection)
Q: ¿Te pidieron usar alguna estructura de programación en particular?

A: No sé / Lo más simple



Q: ¿En qué etapa estás del trabajo?

A: Recién empiezo, solo tengo el diagrama

Perfecto. La forma más intuitiva y sencilla de pasar un diagrama de estados a código C es usando la estructura switch-case. Es la técnica estándar y la más fácil de leer y depurar.

Para entender cómo funciona, imaginemos el ejemplo más clásico: un molinete de subte. Tiene solo dos estados (Bloqueado y Desbloqueado) y dos eventos (Poner Moneda y Empujar).

Antes de ver el código, podés interactuar con este simulador para ver cómo los eventos (los botones) cambian el estado actual del sistema, que es exactamente lo que va a hacer tu programa en C:


Cómo traducirlo a C (Plantilla base)
La clave para que el código sea idéntico a tu diagrama es usar enum (enumeraciones). Esto te permite usar palabras en lugar de números sueltos, haciendo que el código se lea como un mapa.

Acá tenés la estructura básica. Podés copiarla y cambiar los nombres por los de tu propio Trabajo Práctico:

C
#include <stdio.h>

// 1. Definimos los estados (los círculos de tu diagrama)
typedef enum {
    ESTADO_BLOQUEADO,
    ESTADO_DESBLOQUEADO
} EstadoMolinete;

// 2. Definimos los eventos (las flechas/condiciones que te hacen cambiar de estado)
typedef enum {
    EVENTO_MONEDA,
    EVENTO_EMPUJAR,
    EVENTO_NADA
} Evento;

int main() {
    // 3. Inicializamos la máquina en su estado inicial
    EstadoMolinete estadoActual = ESTADO_BLOQUEADO;
    Evento eventoNuevo = EVENTO_NADA; 

    // Bucle principal continuo
    while(1) {
        
        // Aquí iría el código que lee botones, sensores o el teclado de la PC
        // eventoNuevo = leer_sensor(); 
        
        // 4. La Máquina de Estados: Un switch principal con un case por cada estado
        switch(estadoActual) {
            
            case ESTADO_BLOQUEADO:
                // ¿Qué pasa si estoy bloqueado y ocurre un evento?
                if (eventoNuevo == EVENTO_MONEDA) {
                    printf("Moneda detectada. Abriendo...\n");
                    estadoActual = ESTADO_DESBLOQUEADO; // Transición de estado
                } 
                else if (eventoNuevo == EVENTO_EMPUJAR) {
                    printf("Alarma! No puedes pasar.\n");
                    // Me quedo en el mismo estado, no hay transición
                }
                break;

            case ESTADO_DESBLOQUEADO:
                // ¿Qué pasa si estoy desbloqueado y ocurre un evento?
                if (eventoNuevo == EVENTO_EMPUJAR) {
                    printf("Persona pasó. Cerrando...\n");
                    estadoActual = ESTADO_BLOQUEADO; // Transición de estado
                } 
                else if (eventoNuevo == EVENTO_MONEDA) {
                    printf("Ya está abierto, gracias por la donación.\n");
                    // Me quedo en el mismo estado
                }
                break;
        }
        
        eventoNuevo = EVENTO_NADA; // Limpiamos el evento procesado
    }

    return 0;
}
Las 3 reglas de oro para armar tu código:
Un case por cada círculo: Cada estado de tu diagrama debe ser un case dentro del switch principal.

Un if por cada flecha: Dentro de cada estado, usás condicionales if para evaluar qué evento ocurrió y ver si tenés que cambiar la variable estadoActual.

No ejecutar lógica fuera del switch: Todo lo que la máquina "hace" (prender un LED, imprimir un texto) debe ocurrir dentro de los case al momento de hacer la transición.

¿Querés que armemos el código de tu diagrama?

Sí
Dame como ejemplo interactivo el diagrama de estados del debounce de un switch

Dame como ejemplo interactivo el diagrama de estados del debounce de un switch. Con maquinas de estado

¡Elegiste un ejemplo fantástico! El debounce (o antirrebote) es el ejemplo perfecto de por qué necesitamos máquinas de estado: un simple if no puede distinguir un rebote eléctrico de una pulsación real.

El siguiente simulador te muestra exactamente cómo un interruptor mecánico "rebota" antes de estabilizarse, y cómo una máquina de estados de solo cuatro pasos filtra ese ruido para generar una señal limpia:


Análisis del Diagrama de Estados (Debounce)
Para convertir esto en código C para tu TP, primero debemos entender los cuatro estados fundamentales que viste en el simulador:

ESTADO_ABIERTO (IDLE): El interruptor no está presionado. Estamos esperando.

Evento: Detectamos una señal baja (pulsación).

Acción: No confirmamos la pulsación todavía. Iniciamos un temporizador y pasamos al siguiente estado.

ESTADO_FILTRANDO_CIERRE (ESPERA): Vimos algo, pero podría ser ruido.

Lógica: Si la señal vuelve a subir antes de que el temporizador termine, era ruido. Volvemos a ESTADO_ABIERTO.

Acción: Si el temporizador termina y la señal sigue baja, ¡es una pulsación real! Registramos el evento "BOTÓN PRESIONADO" y pasamos a ESTADO_CERRADO.

ESTADO_CERRADO (ESTABLE): El botón está definitivamente presionado.

Evento: Detectamos una señal alta (suelta).

Acción: Iniciamos el temporizador de nuevo y pasamos a filtrar la apertura.

ESTADO_FILTRANDO_APERTURA (ESPERA): Estamos esperando a ver si realmente soltaron el botón o si fue otro rebote.

Lógica: Si la señal baja de nuevo, volvemos a ESTADO_CERRADO.

Acción: Si el temporizador termina y la señal sigue alta, confirmamos que se soltó. Registramos el evento "BOTÓN LIBERADO" y volvemos a ESTADO_ABIERTO.

Esqueleto en C para tu TP
Este es el esqueleto de cómo implementarías esta lógica exacta en C. Es una estructura profesional que separa la máquina de estados de la lectura física del hardware:

C
#include <stdio.h>
#include <stdbool.h>

// Definimos los 4 estados del diagrama
typedef enum {
    ESTADO_ABIERTO,
    ESTADO_FILTRANDO_CIERRE,
    ESTADO_CERRADO,
    ESTADO_FILTRANDO_APERTURA
} EstadoDebounce;

// Estructura para encapsular los datos de la máquina
typedef struct {
    EstadoDebounce estadoActual;
    unsigned long tiempoInicial; // Para medir el tiempo de debounce
    bool salidaLimpia;          // El estado final, ya filtrado
} MaquinaDebounce;

// Constante: Tiempo de debounce (ej: 50ms)
const unsigned long TIEMPO_DEBOUNCE_MS = 50;


// --- FUNCIONES MOCK DE HARDWARE (Simulan la lectura) ---
// En tu TP, estas leerían registros de un microcontrolador (ej: GPIO_Read)
bool leer_pin_fisico_con_rebote() {
    // Por simplicidad para el ejemplo en PC, siempre devuelve 1 (abierto)
    // Deberías cambiar esto por la lectura real del botón.
    return 1; 
}

unsigned long obtener_tiempo_actual_ms() {
    // Por simplicidad, implementa una función que devuelva ms (ej: millis() en Arduino)
    // Para PC, este ejemplo no funcionará sin una librería de tiempo.
    return 0; 
}
// ----------------------------------------------------


// --- FUNCIÓN PRINCIPAL DE LA MÁQUINA DE ESTADOS ---
void actualizar_maquina_debounce(MaquinaDebounce *maquina) {
    bool pinFisico = leer_pin_fisico_con_rebote();
    unsigned long tiempoAhora = obtener_tiempo_actual_ms();

    switch (maquina->estadoActual) {

        case ESTADO_ABIERTO:
            if (pinFisico == 0) { // Detectamos pulsación
                maquina->tiempoInicial = tiempoAhora;
                maquina->estadoActual = ESTADO_FILTRANDO_CIERRE;
            }
            break;

        case ESTADO_FILTRANDO_CIERRE:
            if (pinFisico == 1) { // Fue un rebote rápido
                maquina->estadoActual = ESTADO_ABIERTO;
            } 
            else if ((tiempoAhora - maquina->tiempoInicial) >= TIEMPO_DEBOUNCE_MS) {
                // Confirmamos pulsación
                maquina->salidaLimpia = 0; // El botón está PRESIONADO
                printf("EVENTO: Boton Presionado (Filtrado)\n");
                maquina->estadoActual = ESTADO_CERRADO;
            }
            break;

        case ESTADO_CERRADO:
            if (pinFisico == 1) { // Intentan soltarlo
                maquina->tiempoInicial = tiempoAhora;
                maquina->estadoActual = ESTADO_FILTRANDO_APERTURA;
            }
            break;

        case ESTADO_FILTRANDO_APERTURA:
            if (pinFisico == 0) { // Volvió a bajar (rebote o ruido)
                maquina->estadoActual = ESTADO_CERRADO;
            }
            else if ((tiempoAhora - maquina->tiempoInicial) >= TIEMPO_DEBOUNCE_MS) {
                // Confirmamos que se soltó
                maquina->salidaLimpia = 1; // El botón está LIBERADO
                printf("EVENTO: Boton Liberado (Filtrado)\n");
                maquina->estadoActual = ESTADO_ABIERTO;
            }
            break;
    }
}

int main() {
    // Inicializar la máquina
    MaquinaDebounce miBoton = {ESTADO_ABIERTO, 0, 1}; // Empieza abierto y estable

    printf("Iniciando escaneo de boton (Simulacion)...\n");

    // Bucle principal (como el loop() de un micro)
    while (1) {
        // En una PC, este bucle corre demasiado rápido y no tenemos
        // control directo del hardware, pero esta es la estructura que
        // usarías en un sistema embebido real.
        actualizar_maquina_debounce(&miBoton);
        
        // Aquí podrías usar miBoton.salidaLimpia para tu lógica principal
    }

    return 0;
}
Por qué esta estructura es buena para tu TP:
Usa typedef enum: Hace que el switch principal sea legible con nombres claros (ESTADO_FILTRANDO_CIERRE) en lugar de números mágicos (0, 1, 2).

Usa una struct: Encapsula todos los datos relevantes del botón. Si quisieras debouncar cinco botones, solo necesitás crear cinco instancias de la estructura MaquinaDebounce y llamar a la función de actualización cinco veces.

Encapsulamiento: La lógica de tiempo y transición está contenida, y el resto del programa solo interactúa con miBoton.salidaLimpia.
