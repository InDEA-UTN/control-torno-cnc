# Caracterización del torno

**Entregable de la etapa 1.** Todo lo que se sabe del torno, cada dato con su fuente: una foto,
una medición o una prueba de continuidad. Lo que no se pudo verificar se marca como tal, no se
completa con lo que "suele ser".

Las fotos van en [`../hardware/fotos/`](../hardware/fotos/), con nombres que digan qué muestran
(`placa_drivers_frente.jpg`, no `IMG_4031.jpg`).

## Datos generales

| Dato | Valor | Fuente |
|------|-------|--------|
| Marca / modelo | Marca: Degem Systems / Modelo: TP-LT CNC Lathe | [foto](./imagenes/equipo_frontal.jpg) |
| Alimentación de red | 220V 50Hz | [foto](./imagenes/motor_base_torno.jpeg)|
| Distancia entre puntos | *(completar)* | |
| Volteo sobre la bancada | *(completar)* | |
| Ejes motorizados | *(completar)* | |

## Motores de los ejes

| Eje | Tipo (paso a paso / servo) | Datos de placa | Pasos por vuelta | Paso del husillo de avance (mm) | Fuente |
|-----|----------------------------|----------------|------------------|---------------------------------|--------|
| X | Tipo paso a paso| | | | |
| Z | *(completar)* | | | | |

## Drivers

¿Placas separadas con entradas STEP/DIR/ENABLE, o una placa propietaria con lógica propia? Modelo,
tensión de alimentación, corriente configurada, microstepping configurado y cómo se configura.

*(completar)*

## Husillo

| Dato | Valor | Fuente |
|------|-------|--------|
| Tipo de motor | *(completar)* | |
| Control de velocidad (fija / PWM / relé / manual) | *(completar)* | |
| Rango de RPM | *(completar)* | |
| Encoder o sensor de índice | *(completar)* | |

## Finales de carrera, home y parada de emergencia

Qué sensores hay, dónde están, de qué tipo son (mecánicos, inductivos, ópticos), si son
normalmente abiertos o cerrados, y a qué pin llegan.

Hay tres finales de carrera, pero uno de ellos actua como un mecanismo de seguridad adicional al de la parada de emergencia. Los sensor restantes indican el limite de los ejes X y Z.

*(completar)*

## Fuente de alimentación

Tensiones, corrientes, qué alimenta cada salida.

Segun la placa del motor de la base por donde se deposita el material. El equipo trabaja con 220V, con una corriente de 0.2A.


*(completar)*

## Mapa de pines del DB25

La columna **Estándar** es la asignación del puerto paralelo de una PC (SPP). Las máquinas CNC
viejas no respetan ese uso —usan los pines como salidas y entradas genéricas—, así que solo sirve
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
| 18–25 | Masa | — | | | |

Si hay optoacopladores o buffers entre el conector y los drivers, anotar cuáles y dónde, y si
invierten la señal.

## Esquema

Esquema del cableado relevado, del DB25 a cada driver, motor y sensor. Puede ser a mano y
fotografiado mientras se releva; el definitivo, en KiCad dentro de [`../hardware/`](../hardware/).

*(completar)*

## Conclusión de la etapa

¿Es STEP/DIR simple, o aparece alguno de los riesgos del [plan](plan.md#etapa-1--caracterizar-el-torno-12-semanas)
(servos con lazo cerrado propietario, placa con lógica de movimiento propia)? ¿Qué se conserva del
torno original y qué se reemplaza?

*(completar)*
