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

## Organización

| Ruta | Contenido |
|------|-----------|
| [`fotos/`](fotos/) | Fotos del torno y de su electrónica original (etapa 1) y del montaje nuevo. |
| `placa_interfaz/` | Proyecto KiCad de la placa, cuando exista. |

## Lista de materiales

*(Completar cuando esté elegido el microcontrolador y cerrada la etapa 1: microcontrolador, driver
y motor para las pruebas de banco de las etapas 2 y 3, buffers u optoacopladores, conectores.)*
