# Referencias

## Para empezar (etapa 0)

- **David Austin**, *Generate stepper-motor speed profiles in real time*, Embedded Systems
  Programming, 2005. El cálculo de la rampa de aceleración que se implementa en la etapa 3.
- **Manual de un driver paso a paso típico**: A4988, DRV8825 (Texas Instruments) o TB6600. Entradas
  STEP/DIR/ENABLE, tiempos mínimos de pulso, microstepping y ajuste de corriente.

## Firmware

- **Grbl** — https://github.com/gnea/grbl. En particular `stepper.c` y `planner.c`, que se leen en
  la etapa 4.
- **grblHAL** — https://github.com/grblHAL, y su wiki. Verificar en la plataforma elegida el soporte
  de torno y de roscado con encoder del husillo.
- **LinuxCNC** — https://linuxcnc.org/docs/ — la alternativa con placa Mesa. Su documentación de
  G-code, de HAL (la capa de abstracción de hardware) y de torneado sirve aunque no se use.

## Hardware

- Manuales de referencia del microcontrolador elegido: temporizadores y **modo encoder** de los
  timers del STM32, o el equivalente del i.MX RT1062 de la Teensy 4.1.
- Horowitz y Hill, *The Art of Electronics*.
- Ott, *Electromagnetic Compatibility Engineering*. Masas, blindaje y ruido: lo que hace que una
  máquina que andaba en la mesa falle cuando se enchufa el husillo.
