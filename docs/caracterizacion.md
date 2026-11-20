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

| Eje | Tipo (paso a paso / servo) | Datos de placa | Pasos por vuelta | Paso del husillo de avance (mm) | Estado funcional | Fuente |
|-----|----------------------------|----------------|------------------|---------------------------------|------------------|--------|
| X | Paso a paso bipolar | MAE HY 200 1713 0033 BX04 (0.33 A, NEMA 17) | 200 (1.8°/paso) | **4,0 mm** *(deducido de calibración previa)* | **Operativo al 100%** (desplazamientos precisos validados) | [datasheet](../hardware/HY200-1713-0033-BX04.pdf), [EEPROM](../firmware/firmware_eeprom.hex) y ensayos empíricos |
| Z | Paso a paso bipolar | *(mismo modelo MAE HY200 o equivalente)* | 200 (1.8°/paso) | **4,0 mm** *(deducido de calibración previa)* | **Electrónica operativa / Atasco mecánico** (requiere ajuste de guías, colisas o tuerca de husillo) | [foto](../hardware/fotos/torno_frontal.jpeg), [EEPROM](../firmware/firmware_eeprom.hex) y ensayos empíricos |

> [!NOTE]
> **Deducción cinemática del paso de husillo:** 
> Los motores MAE HY200 poseen 200 pasos/vuelta ($1,8^\circ/\text{paso}$). Con los controladores L297 polarizados fijamente en modo **medio paso (*half-step*)**, el sistema requiere $200 \times 2 = 400\text{ pulsos/vuelta}$. 
> Al contrastar con la calibración activa extraída de la memoria EEPROM del Arduino previo (`$0 = 100.0` y `$2 = 100.0` pasos/mm):
> $$\text{Paso del husillo} = \frac{400\text{ pulsos/vuelta}}{100\text{ pulsos/mm}} = \mathbf{4{,}0\text{ mm/vuelta}}$$
> Asumiendo acoplamiento directo 1:1, ambos ejes cuentan con tornillos de avance de **4 mm de paso**. Pendiente de ratificación física directa con calibre.

## Drivers

El torno cuenta con una **placa unificada original de Degem Systems** que implementa control por **STEP / DIR estándar**. No posee microcontrolador con firmware propietario para la cinemática:

* **Controlador de pasos (translator / indexer):** Dos circuitos integrados **ST L297** (uno por cada eje, X y Z). Cada L297 recibe directamente:
  * Pin 18 (`CLOCK` / STEP): un pulso equivale a un paso.
  * Pin 17 (`CW/CCW` / DIR): define el sentido de giro.
  * Pin 10 (`ENABLE`): habilita las salidas hacia el driver.
  * Pin 19 (`HALF/FULL`): conmuta entre paso completo y medio paso (*half-step*).
  * Control de corriente mediante corte por chopper PWM interno censando sobre resistencias de potencia.
* **Etapa de potencia (drivers):** Dos circuitos integrados **ST L298N** (puente H dual, montados sobre el disipador de aluminio principal). Conmutan la corriente continua hacia las 4 fases de los motores paso a paso bipolares.
* **Aislación y buffers de entrada:** Las señales de control STEP y DIR provenientes del puerto DB25 ingresan mediante cables apantallados a conectores dedicados en la placa (ej. **J3**), pasando por optoacopladores dobles **Toshiba TLP624-2** (un integrado por eje, aislando galvánicamente CLK y DIR). Las señales luego son acondicionadas por el buffer **74HCT244** y las compuertas de seguridad antes de ingresar a los pines 18 (`CLOCK`) y 17 (`CW/CCW`) de los **L297**.
* **Habilitación y modo de paso:** Los pines 10 (`ENABLE`) y 19 (`HALF/FULL`) de los L297 no se comandan desde el DB25. Se encuentran polarizados por hardware en la placa madre (+5V permanente para mantener torque de retención y modo medio paso fijo).
* **Compuertas de seguridad / interlock:** Circuito integrado **M74HCT08B1** (4 compuertas AND de 2 entradas). Implementa lógica cableada de seguridad (por ejemplo, habilitar pulsos únicamente si no hay fin de carrera activado o guarda abierta).
* **Bomba de carga / Watchdog del husillo:** Circuito integrado **74HC123** (doble multivibrador monoestable redisparable). Recibe el tren de pulsos desde el Pin 1 del DB25 (conector J8), manteniendo la salida activa mientras la frecuencia persista; si los pulsos cesan, el monoestable cae tras ~1 s, cortando el relé Celduc.
* **Driver de periféricos auxiliares:** Circuito integrado **ULN2801N** (arreglo de 8 transistores Darlington). Amplifica corriente para accionar el relé de estado sólido del husillo y señales de control.
* **Microstepping configurado:** El L297 admite únicamente **paso completo (1/1)** o **medio paso (1/2)** (fijado por hardware en placa).
* **Restricción temporal en entradas STEP (ancho de pulso):** Los optoacopladores de entrada **Toshiba TLP624-2** son de tipo fototransistor, caracterizados por tiempos de conmutación relativamente lentos ($t_r, t_f \approx 20\text{–}50\ \mu\text{s}$). Como se confirmó en la ingeniería inversa del control previo ([`firmware/firmware_eeprom.hex`](../firmware/firmware_eeprom.hex)), el ancho de pulso STEP debe configurarse a un mínimo de **$100\ \mu\text{s}$** (`$3 = 100`) para garantizar el disparo confiable del pin 18 de los L297. A la velocidad rápida de $900\text{ mm/min}$ ($15\text{ mm/s}$), la frecuencia de paso requerida es de solo $1{,}5\text{ kHz}$ (período de $666\ \mu\text{s}$), por lo que el pulso de $100\ \mu\text{s}$ opera con holgura.

## Husillo

| Dato | Valor | Fuente |
|------|-------|--------|
| Tipo de motor | Motor de inducción monofásico de CA 220V (Lafert, Made in Italy, 1~ IEC 34-1, tipo 56 C 2) | [foto](../hardware/fotos/motor_base_torno.jpeg) |
| Datos eléctricos de placa | 220 VCA - 50 Hz - 1,2 A - ~0,11 kW - $\cos\varphi = 0,89$ - Capacitor: $4\ \mu\text{F} / 450\text{ V}$ | [foto](../hardware/fotos/motor_base_torno.jpeg) |
| Velocidad nominal motor | ~2610 – 2800 RPM (2 polos, 50 Hz) | [foto](../hardware/fotos/motor_base_torno.jpeg) |
| Transmisión | Polea y correa plana con rueda tensora hacia el eje del husillo | [foto](../hardware/fotos/motor_base_torno.jpeg) |
| Control de velocidad | Digital ON / OFF comandado por **Bomba de Carga (*Charge Pump / Watchdog*)** hacia Relé de Estado Sólido (SSR) | Ensayos empíricos y mediciones con multímetro |
| Modelo del actuador | SSR Celduc France SC841110 (Salida: 230 VCA / 10 A - Entrada: 4-30 VCC) | [foto](../hardware/fotos/equipo_parte_trasera2.jpeg) |
| Señal de control requerida | **Onda cuadrada continua de ~1 kHz a 2 kHz (Pin 1 DB25 / J8)**. Por seguridad, la placa madre desacopla en alterna (bloquea continua DC); un nivel continuo de 5V o 0V es ignorado. Requiere pulsos permanentes para mantener cargado el capacitor que alimenta al SSR. Al cesar los pulsos, se descarga en ~1 s y el motor se apaga. | Ensayos funcionales y oscilación por Timer 0 de Grbl |
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

La alimentación interna está provista por un transformador **A. Kirschner Ltd (0490 02/97)** y etapas de rectificación y regulación dedicadas:

* **Entrada de red:** 220 VCA 50 Hz. El transformador cuenta con dos primarios de 115V conectados en serie (230 VCA nominales).
* **Alimentación de potencia (Motores):** Secundario de **18 VCA (8 A)** (pines 5 y 7). Rectificado mediante un puente rectificador **PBU605** (6 A, 600 V) y filtrado con capacitores electrolíticos, entregando aproximadamente **~25 VCC no regulada** para alimentar la etapa de potencia de los drivers L298N. (El bobinado de 16 VCA no se utiliza).
* **Alimentación lógica (+5V):** Secundario de **7.5 VCA (1.5 A)** (pines 8 y 7), rectificado y regulado mediante un **L7805CV** a **+5 VCC (1.5 A)** para alimentar la lógica digital (L297, 74HCT244, M74HCT08B1, ULN2801N y sensores).

## Mapa de pines del DB25

> [!NOTE]
> **Alcance del relevamiento del DB25:** Al armar inicialmente el repositorio, se asumió que la migración debía comenzar desde cero, por lo que se planteó relevar la totalidad de los 25 pines del conector paralelo. Sin embargo, al hallar el controlador **Arduino Nano** ya cableado a la máquina, se verificaron únicamente los pines efectivamente utilizados por el sistema (ejes X/Z, husillo, fines de carrera y masa). Los pines restantes marcados como *(pendiente)* no se continuaron relevando por no ser necesarios para la puesta en marcha ni para la operación actual.

La columna **Estándar** es la asignación del puerto paralelo de una PC (SPP). Las máquinas CNC
viejas no respetan ese uso -usan los pines como salidas y entradas genéricas-, así que solo sirve
para saber **qué dirección tiene cada pin del lado de la PC**. Lo que importa son las columnas de la
derecha, y se completan con medición.

| Pin | Estándar (desde la PC) | Dirección | Función en el torno | Destino (cable / placa / borne) | Verificado con |
|-----|------------------------|-----------|---------------------|---------------------------------|----------------|
| 1 | /Strobe | Salida | **Control de Husillo (ON/OFF)** | Ficha J8 (cable apantallado) -> Etapa de control SSR Celduc | Inspección física / cableado |
| 2 | D0 | Salida | **STEP Eje Z** (CLOCK) | Conector placa (J3) -> Optoacoplador TLP624-2 -> L297 Eje Z (Pin 18) | Relevamiento de placa / continuidades |
| 3 | D1 | Salida | **DIR Eje Z** (CW/CCW) | Conector placa (J3) -> Optoacoplador TLP624-2 -> L297 Eje Z (Pin 17) | Relevamiento de placa / continuidades |
| 4 | D2 | Salida | **STEP Eje X** (CLOCK) | Conector placa -> Optoacoplador TLP624-2 -> L297 Eje X (Pin 18) | Relevamiento de placa / continuidades |
| 5 | D3 | Salida | **DIR Eje X** (CW/CCW) | Conector placa -> Optoacoplador TLP624-2 -> L297 Eje X (Pin 17) | Relevamiento de placa / continuidades |
| 6 | D4 | Salida | **Desconectado (NC)** | Sin conexión | Continuidad física / inspección |
| 7 | D5 | Salida | *(pendiente)* | | |
| 8 | D6 | Salida | *(pendiente)* | | |
| 9 | D7 | Salida | *(pendiente)* | | |
| 10 | /Ack | Entrada | **Fin de carrera Eje Z** | Conector J11 / microswitch Z hacia PC/Arduino | Verificación física / pinout |
| 11 | Busy | Entrada | *(pendiente)* | | |
| 12 | Paper Out | Entrada | **Fin de carrera Eje X** | Conector J10 / microswitch X hacia PC/Arduino | Verificación empírica con multímetro |
| 13 | Select | Entrada | *(pendiente)* | | |
| 14 | /Auto Feed | Salida | *(pendiente)* | | |
| 15 | /Error | Entrada | *(pendiente)* | | |
| 16 | /Init | Salida | *(pendiente)* | | |
| 17 | /Select In | Salida | *(pendiente)* | | |
| 18 | Masa | — | **Masa lógica (GND)** | GND común de la placa | Continuidad física |
| 19 | Masa | — | **Masa lógica (GND)** | GND común de la placa | Continuidad física |
| 20 | Masa | — | **Desconectado (NC)** | Sin conexión | Continuidad física / inspección |
| 21 | Masa | — | *(pendiente de verificar GND o NC)* | | |
| 22 | Masa | — | **Desconectado (NC)** | Sin conexión | Continuidad física / inspección |
| 23 | Masa | — | **Desconectado (NC)** | Sin conexión | Continuidad física / inspección |
| 24 | Masa | — | **Desconectado (NC)** | Sin conexión | Continuidad física / inspección |
| 25 | Masa | — | **Desconectado (NC)** | Sin conexión | Continuidad física / inspección |

Las entradas de STEP y DIR provenientes del DB25 cuentan con aislación galvánica mediante optoacopladores dobles **Toshiba TLP624-2** y acondicionamiento mediante un buffer **74HCT244** antes de ingresar a los pines de control de los L297.

## Relevamiento del control previo (Arduino / Grbl)

En el torno se halló un microcontrolador ATmega328P (Arduino Uno / Nano) utilizado previamente para comandar la máquina. Se extrajo tanto la memoria de programa Flash ([`firmware/firmware_extraido.hex`](../firmware/firmware_extraido.hex)) como la memoria no volátil EEPROM ([`firmware/firmware_eeprom.hex`](../firmware/firmware_eeprom.hex)). El análisis del código binario y la decodificación de las estructuras persistentes confirmaron lo siguiente:

* **Firmware:** **Grbl v0.8c** (compilado para ATmega328P a 16 MHz, 9600 baudios, con bootloader Optiboot en `0x7E00`).
* **Pin de control de husillo:** La señal de encendido de husillo (*Spindle Enable/PWM*) está asignada al pin **D11** del Arduino (conmuta a ~4,3 V con `M3` y cae a 0 V con `M5`, configuración típica con `VARIABLE_SPINDLE` habilitado).
* **Estado de la calibración:** La EEPROM cuenta con datos válidos con suma de comprobación (*checksum*) verificada (`0x1A`), evidenciando parámetros configurados y utilizados en la máquina.

### Parámetros extraídos de la EEPROM

| Parámetro Grbl | Valor en EEPROM | Significado físico / Relevancia para el torno |
| :--- | :--- | :--- |
| `$0` | `100.0` paso/mm | Resolución Eje X (transversal) |
| `$2` | `100.0` paso/mm | Resolución Eje Z (longitudinal) |
| `$3` | `100` µs | Ancho de pulso STEP (adaptado a los optoacopladores lentos TLP624-2) |
| `$4` | `350.0` mm/min | Avance de corte de referencia (*feedrate*) |
| `$5` | `900.0` mm/min | Velocidad máxima en desplazamientos rápidos (*seek*) |
| `$6` | `192` (`0b11000000`) | Máscara de inversión de sentido: Eje Z invertido respecto de X |
| `$8` | `10.0` mm/s² | Aceleración ($36.000\text{ mm/min²}$) |
| `flags` | `14` (`0b00001110`) | Habilitación de límites por hardware (*hard limits*) e inversión de señal Enable |

Esta configuración ratifica que los husillos de ambos ejes tienen un avance de **4 mm/vuelta** (considerando el motor de 200 pasos y el modo de medio paso del L297: $\frac{400\text{ pulsos/rev}}{100\text{ pulsos/mm}} = 4\text{ mm}$).

## Esquema

Esquema del cableado relevado, del DB25 a cada driver, motor y sensor. Puede ser a mano y
fotografiado mientras se releva; el definitivo, en KiCad dentro de [`../hardware/`](../hardware/).

> [!NOTE]
> **Estado del diseño de hardware (`placa_interfaz/`):** Este apartado y el esquemático definitivo en KiCad quedaron sin desarrollar porque, al encontrar el **Arduino Nano** ya instalado y cableado al DB25, se optó por recuperar y adaptar esa controladora por firmware en lugar de diseñar una placa de interfaz desde cero.

## Conclusión de la etapa

¿Es STEP/DIR simple, o aparece alguno de los riesgos del [plan](plan.md#etapa-1--caracterizar-el-torno-12-semanas)
(servos con lazo cerrado propietario, placa con lógica de movimiento propia)? ¿Qué se conserva del
torno original y qué se reemplaza?

* **¿Es STEP/DIR simple?** **SÍ.** La electrónica original se basa en el par clásico **L297 + L298N**. Los motores se controlan directamente mediante trenes de pulsos (STEP) y niveles de dirección (DIR) con lógica TTL de 5V.
* **Riesgo de hardware cerrado:** Queda **descartado**. No existe un procesador propietario ni protocolo serie que interfiera en la cinemática.
* **Estrategia de modernización:** Se puede conservar íntegramente la etapa de potencia actual (fuente, drivers L298N, L297 y relé de estado sólido del husillo) y reemplazar únicamente la PC con puerto paralelo por una controladora moderna (Teensy 4.1 o STM32 con grblHAL). Solo se requerirá acondicionar las señales de 3.3V del microcontrolador a los 5V que espera el buffer 74HCT244 de la placa, y proveer la señal de ~1 a 2 kHz para la bomba de carga del husillo.

## Estado de entrega y transferencia al Laboratorio 3D de la Universidad (UTN)

El sistema queda caracterizado, validado y operativo con la controladora intermedia (Arduino Nano corriendo Grbl v0.8c adaptado con `grbl_v08c_remapeado.hex`):
1. **Eje X (Transversal):** 100% operativo y calibrado (100 pasos/mm, husillo paso 4 mm, motor MAE HY200 en medio paso). Responde con suavidad y precisión a comandos G-code.
2. **Eje Z (Longitudinal):** Cadena de control electrónico 100% validada (señales STEP/DIR llegan a los transistores de potencia). **Pendiente para el laboratorio:** Desarme, limpieza, lubricación y desajuste de colisas mecánicas / tuerca de avance debido a fricción/atasco mecánico que frena el motor.
3. **Husillo (Motor AC 220V):** 100% operativo comandado por software (`M3 S1000` enciende, `M5` apaga). Se resolvió la ingeniería inversa de la bomba de carga (*Charge Pump*), generando una onda cuadrada continua de ~2 kHz desde el microcontrolador.
4. **Seguridad:** Enclavamiento por hardware (*Hardware Interlock* con compuertas AND `M74HCT08B1`) verificado: la máquina inhibe el paso de pulsos si la guarda de protección (J14) está abierta.
5. **Firmware:** Código fuente documentado en [`../firmware/grbl_v08c_torno/`](../firmware/grbl_v08c_torno/) y binario listo para flashear en [`../firmware/grbl_v08c_remapeado.hex`](../firmware/grbl_v08c_remapeado.hex). Respaldos originales intactos.
