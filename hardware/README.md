# hardware/

El desarrollo propio del proyecto: la **placa de interfaz** entre el microcontrolador y el torno.

## Qué tiene que resolver la placa

- **Niveles lógicos.** El puerto paralelo original es de 5 V; el microcontrolador, de 3,3 V. Las
  salidas hacia los drivers pasan por buffers 74HCT (que aceptan 3,3 V como nivel alto y entregan
  5 V) u optoacopladores. Las entradas desde el torno nunca llegan directo a un pin de 3,3 V.
- **Entradas protegidas.** Finales de carrera, home y parada de emergencia: filtrado, protección
  contra sobretensión y, preferentemente, aislación.
- **Parada de emergencia por hardware.** Corta la potencia de los drivers y del husillo con
  contactos normalmente cerrados. El firmware se entera, pero no es quien corta.
- **Ruido.** Masas separadas entre lógica y potencia, optoacopladores y cable blindado desde el
  principio, no como arreglo cuando aparezcan fallas.

El diseño depende de lo que salga de la [caracterización](../docs/caracterizacion.md): hasta
cerrar la etapa 1 no se diseña nada.

## Hojas de Datos de Componentes (Datasheets)

A continuación se listan las hojas de datos de los circuitos integrados y componentes electromecánicos relevados en la máquina original:

| Archivo | Componente / Integrado | Función en el torno |
| :--- | :--- | :--- |
| [`HY200-1713-0033-BX04.pdf`](HY200-1713-0033-BX04.pdf) | **MAE HY 200 1713 0033 BX04** | Motor paso a paso bipolar (NEMA 17, 200 pasos/rev, 0.33 A, ejes X y Z). |
| [`DS_l297-1.pdf`](DS_l297-1.pdf) | **ST L297** | Controlador de pasos y conmutación de fases (CLOCK, DIR, Chopper PWM). |
| [`DS_l298-1.pdf`](DS_l298-1.pdf) | **ST L298N** | Driver de potencia puente H dual (alimentación de bobinas de motores). |
| [`74HC123.PDF`](74HC123.PDF) | **74HC123** | Doble monoestable redisparable (circuito de Bomba de Carga / Watchdog del husillo). |
| [`DS_uln2801a.pdf`](DS_uln2801a.pdf) | **ULN2801A** | Arreglo de transistores Darlington (driver de cargas y conmutación auxiliar). |
| [`TLP624_datasheet.pdf`](TLP624_datasheet.pdf) | **Toshiba TLP624-2** | Optoacopladores dobles (aislación galvánica de líneas STEP y DIR). |
| [`M54HCT08.PDF`](M54HCT08.PDF) | **M74HCT08B1** | Cuádruple compuerta AND (enclavamiento de seguridad e interlock de tapa J14). |
| [`SC841110.PDF`](SC841110.PDF) | **Celduc SC841110** | Relé de estado sólido (SSR 230 VCA / 10 A para arranque del motor AC de husillo). |
| [`L7800.PDF`](L7800.PDF) | **L7805CV** | Regulador lineal de tensión (+5 VCC lógica digital). |
| [`PBU601.PDF`](PBU601.PDF) | **PBU605** | Puente rectificador (etapa de potencia de motores ~25 VCC no regulada). |

## Organización

| Ruta | Contenido |
|------|-----------|
| [`fotos/`](fotos/) | Fotos del torno y de su electrónica original (etapa 1) y del montaje nuevo. |
| `placa_interfaz/` | Proyecto KiCad de la placa, cuando exista. |

## Lista de materiales

> [!NOTE]
> **Estado del desarrollo de hardware propio:** La placa de interfaz (`placa_interfaz/`), su diseño en KiCad y esta lista de materiales quedaron sin completar debido a que, tras armar la estructura inicial del repositorio, se encontró un **Arduino Nano** ya instalado y cableado al torno. Al aprovechar y adaptar esa controladora existente mediante firmware, dejó de ser necesario diseñar e implementar una placa de interfaz desde cero en esta etapa.
