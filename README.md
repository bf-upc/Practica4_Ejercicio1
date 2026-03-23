# Práctica 4 – Ejercicio 1: Multitarea en ESP32 con FreeRTOS

## Descripción

Este proyecto forma parte de la **Práctica 4 de Sistemas Operativos en Tiempo Real**. El objetivo es comprender el funcionamiento de un sistema operativo en tiempo real observando cómo el planificador de FreeRTOS divide el tiempo de CPU entre múltiples tareas concurrentes en un ESP32.

En este ejercicio se crean dos tareas que se ejecutan simultáneamente:
- La tarea principal (`loop`) imprime un mensaje por el puerto serie cada segundo.
- Una tarea adicional (`anotherTask`) imprime un mensaje diferente también cada segundo.

Ambas tareas comparten el tiempo de CPU, demostrando el funcionamiento del planificador de FreeRTOS.

---

## Hardware necesario

- Placa **ESP32** (cualquier variante compatible)
- Cable USB para programación y monitor serie

---

## Software y dependencias

- [PlatformIO](https://platformio.org/) (IDE o extensión para VS Code)
- Framework: **Arduino** para ESP32
- FreeRTOS (incluido en el framework de Arduino para ESP32)

---

## Estructura del proyecto

```
Practica4_Ejercicio1/
├── src/
│   └── main.cpp        # Código fuente principal
├── include/            # Cabeceras adicionales (vacío por defecto)
├── lib/                # Librerías locales (vacío por defecto)
├── test/               # Tests (vacío por defecto)
└── platformio.ini      # Configuración del proyecto PlatformIO
```

---

## Código

```cpp
void setup()
{
    Serial.begin(112500);

    /* Creamos una nueva tarea */
    xTaskCreate(
        anotherTask,      /* Función de la tarea */
        "another Task",   /* Nombre de la tarea (para depuración) */
        10000,            /* Tamaño del stack (bytes) */
        NULL,             /* Parámetro de la tarea */
        1,                /* Prioridad de la tarea */
        NULL              /* Handle de la tarea */
    );
}

/* La función loop() es invocada por la tarea loopTask del ESP32 Arduino */
void loop()
{
    Serial.println("this is ESP32 Task");
    delay(1000);
}

/* Esta función se invoca cuando se crea anotherTask */
void anotherTask(void * parameter)
{
    for(;;)
    {
        Serial.println("this is another Task");
        delay(1000);
    }
    /* Esta línea nunca se alcanza por ser un bucle infinito */
    vTaskDelete(NULL);
}
```

---

## Explicación del funcionamiento

El ESP32 ejecuta FreeRTOS internamente. Al usar el framework de Arduino, la función `loop()` ya corre dentro de una tarea gestionada por FreeRTOS llamada `loopTask`. En `setup()` creamos una segunda tarea con `xTaskCreate()`.

El planificador de FreeRTOS se encarga de repartir el tiempo de CPU entre ambas tareas. Como las dos tienen la **misma prioridad (1)**, el planificador las alterna equitativamente. Cuando una tarea llama a `delay()`, cede el control al planificador, que puede ejecutar la otra tarea mientras espera.

> **Nota importante:** En FreeRTOS se recomienda usar `vTaskDelay()` en lugar de `delay()` para pausar tareas, ya que `vTaskDelay()` notifica explícitamente al planificador que la tarea puede bloquearse, lo que permite una gestión más eficiente de la CPU.

---

## Salida esperada por el puerto serie

La salida por el puerto serie (a 112500 baudios) mostrará los mensajes de ambas tareas intercalados:

```
this is ESP32 Task
this is another Task
this is ESP32 Task
this is another Task
...
```

El orden exacto puede variar ligeramente dependiendo del planificador, pero ambas tareas se ejecutan aproximadamente cada segundo. Dado que comparten la misma prioridad, FreeRTOS reparte el tiempo de CPU entre ellas de forma equitativa.

---

## Cómo compilar y cargar

1. Abre el proyecto con PlatformIO.
2. Conecta el ESP32 por USB.
3. Ejecuta **Build** y luego **Upload**.
4. Abre el **Monitor Serie** a **112500 baudios** para ver la salida.

---

## Conceptos clave de FreeRTOS utilizados

| Concepto | Descripción |
|---|---|
| `xTaskCreate()` | Crea una nueva tarea y la registra en el planificador |
| `vTaskDelay()` | Pausa la tarea un tiempo determinado, cediendo la CPU |
| `vTaskDelete()` | Elimina una tarea cuando ya no es necesaria |
| Prioridad de tarea | Número que indica la importancia relativa; mayor número = mayor prioridad |
| Planificador (Scheduler) | Componente de FreeRTOS que decide qué tarea se ejecuta en cada momento |

---

## Referencias

- [Repositorio de referencia ESP32 FreeRTOS](https://github.com/uagaviria/ESP32_FreeRtos)
- [Documentación oficial de FreeRTOS](https://www.freertos.org/)
- [Documentación de PlatformIO](https://platformio.org/)