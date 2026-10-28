# Caracterización del torno

**Entregable de la etapa 1.** Todo lo que se sabe del torno, cada dato con su fuente: una foto,
una medición o una prueba de continuidad. Lo que no se pudo verificar se marca como tal, no se
completa con lo que "suele ser".

Las fotos van en [`../hardware/fotos/`](../hardware/fotos/), con nombres que digan qué muestran
(`placa_drivers_frente.jpg`, no `IMG_4031.jpg`).

## Datos generales

| Dato | Valor | Fuente |
|------|-------|--------|
| Marca / modelo | Marca: Degem Systems / Modelo: TP-LT CNC Lathe | [foto](../hardware/fotos/equipo_frontal.jpeg) |
| Alimentación de red | 220V 50Hz | [foto](../hardware/fotos/motor_base_torno.jpeg) |
| Distancia entre puntos | *(completar)* | |
| Volteo sobre la bancada | *(completar)* | |
| Ejes motorizados | 2 ejes (X y Z) | [foto](../hardware/fotos/torno_frontal.jpeg) |

## Motores de los ejes

| Eje | Tipo (paso a paso / servo) | Datos de placa | Pasos por vuelta | Paso del husillo de avance (mm) | Fuente |
|-----|----------------------------|----------------|------------------|---------------------------------|--------|
| X | Paso a paso bipolar | MAE HY 200 1713 0033 BX04 (0.33 A, NEMA 17) | 200 (1.8°/paso) | *(completar)* | [datasheet](../hardware/HY200-1713-0033-BX04.pdf) |
| Z | Paso a paso bipolar | *(verificar si es el mismo modelo MAE HY200)* | 200 (a confirmar) | *(completar)* | [foto](../hardware/fotos/torno_frontal.jpeg) |

## Drivers

¿Placas separadas con entradas STEP/DIR/ENABLE, o una placa propietaria con lógica propia? Modelo,
tensión de alimentación, corriente configurada, microstepping configurado y cómo se configura.

El torno cuenta con una **placa unificada original de Degem Systems** que implementa control por **STEP / DIR estándar**. No posee microcontrolador con firmware propietario para la cinemática:

* **Controlador de pasos (translator / indexer):** Dos circuitos integrados **ST L297** (uno por cada eje, X y Z). Cada L297 recibe directamente:
  * Pin 18 (`CLOCK` / STEP): un pulso equivale a un paso.
  * Pin 17 (`CW/CCW` / DIR): define el sentido de giro.
  * Pin 10 (`ENABLE`): habilita las salidas hacia el driver.
  * Pin 19 (`HALF/FULL`): conmuta entre paso completo y medio paso (*half-step*).
  * Control de corriente mediante corte por chopper PWM interno censando sobre resistencias de potencia.
* **Etapa de potencia (drivers):** Dos circuitos integrados **ST L298N** (puente H dual, montados sobre el disipador de aluminio principal). Conmutan la corriente continua hacia las 4 fases de los motores paso a paso bipolares.
* **Buffer de entrada del puerto DB25:** Circuito integrado **74HCT244** (Octal buffer / line driver con entradas compatibles TTL 5V). Recibe las señales del conector DB25 (puerto paralelo de la PC), limpiando el ruido y protegiendo las entradas de los L297.
* **Compuertas de seguridad / interlock:** Circuito integrado **M74HCT08B1** (4 compuertas AND de 2 entradas). Implementa lógica cableada de seguridad (por ejemplo, habilitar pulsos únicamente si no hay fin de carrera activado o guarda abierta).
* **Driver de periféricos auxiliares:** Circuito integrado **ULN2801N** (arreglo de 8 transistores Darlington). Amplifica corriente para accionar el relé de estado sólido del husillo y señales de control.
* **Microstepping configurado:** El L297 admite únicamente **paso completo (1/1)** o **medio paso (1/2)**. *(Pendiente verificar si el pin 19 HALF/FULL está cableado fijo a nivel alto/bajo o si se conmuta desde el DB25)*.

## Husillo

| Dato | Valor | Fuente |
|------|-------|--------|
| Tipo de motor | Motor de CA 220V *(a verificar placa del motor)* | [foto](../hardware/fotos/equipo_parte_trasera.jpeg) |
| Control de velocidad | Digital ON / OFF mediante Relé de Estado Sólido (SSR) | Relevamiento de placa |
| Modelo del actuador | SSR Celduc France SC841110 (Salida: 230 VCA / 10 A - Entrada: 4-30 VCC) | [foto](../hardware/fotos/equipo_parte_trasera2.jpeg) |
| Rango de RPM | *(completar según placa del husillo)* | |
| Encoder o sensor de índice | *(completar: verificar si cuenta con sensor óptico/inductivo de 1 pulso por vuelta para roscado)* | |

## Finales de carrera, home y parada de emergencia

El torno cuenta con tres microswitches mecánicos cableados como contactos **Normalmente Abiertos (NO)** que conmutan a masa (GND) al accionarse:

| Sensor / Función | Conector en placa | Tipo de contacto | Estado en reposo | Estado activado | Acondicionamiento en placa |
|---|---|---|---|---|---|
| Límite Eje X | **J10** | Mecánico NO | Abierto (+5V por pull-up) | Cerrado a GND (0V) | Pull-up a +5V + R serie de 6,2 kΩ + filtro RC a compuerta AND |
| Límite Eje Z | **J11** | Mecánico NO | Abierto (+5V por pull-up) | Cerrado a GND (0V) | Pull-up a +5V + R serie de protección + filtro RC a compuerta AND |
| Sensor tapa / guarda | **J14** | Mecánico NO | Abierto (+5V por pull-up) | Cerrado a GND (0V) | Pull-up a +5V; interlock de seguridad / parada |

* **Lógica de seguridad e interlock:** El circuito integrado **M74HCT08B1** (4 compuertas AND) actúa como corte de seguridad por hardware (*hardware interlock*). En reposo, las resistencias de pull-up mantienen las entradas de habilitación en nivel ALTO (+5V / `1` lógico), permitiendo el paso de los pulsos de paso (STEP) hacia los controladores **L297**. Si un eje llega al límite físico o se abre la guarda de protección, la entrada cae a nivel BAJO (0V / `0` lógico), forzando instantáneamente la salida de la compuerta AND a `0` y bloqueando el avance de los motores por hardware, sin depender del microcontrolador ni del software.
* **Protección contra ruido:** Las señales de los sensores no ingresan directo a las compuertas CMOS; pasan por una resistencia en serie de **6,2 kΩ** (medida en J10) y capacitores cerámicos de filtrado a masa, conformando un filtro pasabajos RC que absorbe el ruido electromagnético de los motores y el rebote mecánico del switch.

## Fuente de alimentación

Tensiones, corrientes, qué alimenta cada salida.

La alimentación interna está provista por un transformador **A. Kirschner Ltd (0490 02/97)** y etapas de rectificación y regulación dedicadas:

* **Entrada de red:** 220 VCA 50 Hz. El transformador cuenta con dos primarios de 115V conectados en serie (230 VCA nominales).
* **Alimentación de potencia (Motores):** Secundario de **18 VCA (8 A)** (pines 5 y 7). Rectificado mediante un puente rectificador **PBU605** (6 A, 600 V) y filtrado con capacitores electrolíticos, entregando aproximadamente **~25 VCC no regulada** para alimentar la etapa de potencia de los drivers L298N. (El bobinado de 16 VCA no se utiliza).
* **Alimentación lógica (+5V):** Secundario de **7.5 VCA (1.5 A)** (pines 8 y 7), rectificado y regulado mediante un **L7805CV** a **+5 VCC (1.5 A)** para alimentar la lógica digital (L297, 74HCT244, M74HCT08B1, ULN2801N y sensores).

## Mapa de pines del DB25

La columna **Estándar** es la asignación del puerto paralelo de una PC (SPP). Las máquinas CNC
viejas no respetan ese uso -usan los pines como salidas y entradas genéricas-, así que solo sirve
para saber **qué dirección tiene cada pin del lado de la PC**. Lo que importa son las columnas de la
derecha, y se completan con medición.

| Pin | Estándar (desde la PC) | Dirección | Función en el torno | Destino (cable / placa / borne) | Verificado con |
|-----|------------------------|-----------|---------------------|---------------------------------|----------------|
| 1 | /Strobe | Salida | | | |
| 2 | D0 | Salida | | | |
| 3 | D1 | Salida | | | |
| 4 | D2 | Salida | | | |
| 5 | D3 | Salida | | | |
| 6 | D4 | Salida | | | |
| 7 | D5 | Salida | | | |
| 8 | D6 | Salida | | | |
| 9 | D7 | Salida | | | |
| 10 | /Ack | Entrada | | | |
| 11 | Busy | Entrada | | | |
| 12 | Paper Out | Entrada | | | |
| 13 | Select | Entrada | | | |
| 14 | /Auto Feed | Salida | | | |
| 15 | /Error | Entrada | | | |
| 16 | /Init | Salida | | | |
| 17 | /Select In | Salida | | | |
| 18—25 | Masa | — | Masa lógica / chasis | GND común de la placa | |

Si hay optoacopladores o buffers entre el conector y los drivers, anotar cuáles y dónde, y si
invierten la señal. *(Las entradas de control pasan por el buffer 74HCT244).*

## Esquema

Esquema del cableado relevado, del DB25 a cada driver, motor y sensor. Puede ser a mano y
fotografiado mientras se releva; el definitivo, en KiCad dentro de [`../hardware/`](../hardware/).

*(completar)*

## Conclusión de la etapa

¿Es STEP/DIR simple, o aparece alguno de los riesgos del [plan](plan.md#etapa-1--caracterizar-el-torno-12-semanas)
(servos con lazo cerrado propietario, placa con lógica de movimiento propia)? ¿Qué se conserva del
torno original y qué se reemplaza?

* **¿Es STEP/DIR simple?** **SÍ.** La electrónica original se basa en el par clásico **L297 + L298N**. Los motores se controlan directamente mediante trenes de pulsos (STEP) y niveles de dirección (DIR) con lógica TTL de 5V.
* **Riesgo de hardware cerrado:** Queda **descartado**. No existe un procesador propietario ni protocolo serie que interfiera en la cinemática.
* **Estrategia de modernización:** Se puede conservar íntegramente la etapa de potencia actual (fuente, drivers L298N, L297 y relé de estado sólido del husillo) y reemplazar únicamente la PC con puerto paralelo por una controladora moderna (Teensy 4.1 o STM32 con grblHAL). Solo se requerirá acondicionar las señales de 3.3V del microcontrolador a los 5V que espera el buffer 74HCT244 de la placa.
