# Control moderno para un mini torno CNC

El laboratorio tiene un mini torno CNC (control numérico computarizado) antiguo: se comandaba desde
una PC por **puerto paralelo**, el software original no existe, la marca no tiene registro y no hay
manual. El objetivo es **reemplazarle el control por uno moderno, por USB o Ethernet**, que no
dependa de una PC vieja con puerto paralelo, y dejarlo mecanizando de nuevo.

Proyecto del laboratorio **InDEA** (Investigación y Desarrollo en Electrónica Aplicada), UTN
Facultad Regional Mendoza.

- **Responsable:** Gonzalo Luna
- **Director:** Javier Velez
- **Estado:** Etapa 0 — preparación (ver [`docs/plan.md`](docs/plan.md))

## El problema

Un control de máquina CNC es una cadena. Cada eslabón transforma lo que le llega del anterior:

```
G-code → intérprete → planificador → generador de pulsos → driver → motor → carro
                      (velocidades)   (timer + interrupción)  (STEP/DIR)
```

Alrededor de esa cadena están los **finales de carrera**, el **homing** (la búsqueda del cero de
máquina), la **parada de emergencia**, y el **husillo** con su encoder o sensor de índice.

La idea que sostiene todo lo demás: **un pulso en la entrada STEP del driver es un paso del
motor.** La posición es contar pulsos, y la velocidad es la frecuencia de esos pulsos.

## Estrategia

- **El firmware no se escribe desde cero:** se adapta uno abierto. La referencia es
  [grblHAL](https://github.com/grblHAL), que corre en STM32, Teensy 4.x y ESP32 y tiene soporte de
  torno —y de roscado con encoder en algunas plataformas, **a verificar en la que se elija**—. La
  alternativa es [LinuxCNC](https://linuxcnc.org/) con una placa Mesa.
- **El desarrollo propio está en el hardware:** la placa de interfaz, la adaptación de niveles, las
  protecciones y el cableado al torno.
- **Microcontrolador candidato:** Teensy 4.1 o una STM32 Nucleo. La elección es parte del trabajo.
- **Niveles lógicos:** el puerto paralelo original es de **5 V** y los microcontroladores modernos
  son de **3,3 V**. Se resuelve con buffers 74HCT u optoacopladores, no conectando directo.

Antes de usar grblHAL se construye a mano una versión mínima de lo que hace (etapas 2 y 3): un
motor, un timer y una rampa. No es para reemplazarlo, es para entender qué hace cuando se lo
configura y para saber dónde buscar cuando algo falle.

## Seguridad

Es una máquina con un plato que gira y herramientas de corte. Desde el primer día:

- **La parada de emergencia corta la potencia por hardware**, con contactos normalmente cerrados.
  Nunca depende solo del firmware: un firmware colgado no para nada.
- Nada se prueba con el husillo girando hasta que la parada de emergencia esté instalada y probada.
- Las pruebas de las etapas 2 a 4 se hacen con el motor **fuera del torno** o con el carro lejos de
  los topes, a velocidades bajas primero.
- Antes de desarmar o medir sobre la electrónica original: desenchufada, y con los capacitores de
  la fuente descargados.

## Organización del repositorio

| Ruta | Contenido |
|------|-----------|
| [`docs/plan.md`](docs/plan.md) | Las etapas del proyecto, su criterio de éxito y en cuál estamos. |
| [`docs/conceptos.md`](docs/conceptos.md) | Los seis conceptos clave, explicados con palabras propias (entregable de la etapa 0). |
| [`docs/caracterizacion.md`](docs/caracterizacion.md) | Relevamiento del torno: motores, drivers, husillo, sensores y mapa de pines del DB25. |
| [`docs/mediciones.md`](docs/mediciones.md) | Resultados de las pruebas de cada etapa: frecuencias, rampas, precisión, RPM. |
| [`docs/referencias.md`](docs/referencias.md) | Bibliografía y enlaces. |
| [`hardware/`](hardware/) | Placa de interfaz, esquemas, lista de materiales y fotos del torno. |
| [`firmware/`](firmware/) | Pruebas propias de las etapas 2 y 3 y la configuración de grblHAL. |

## Cómo se trabaja acá

`main` está protegido: los cambios entran por **pull request**. El flujo de trabajo habitual es
crear una rama, subirla, abrir el pull request y pedir revisión a Javier.

```bash
git switch -c descripcion-corta-del-cambio
# ... trabajar, git add, git commit ...
git push -u origin descripcion-corta-del-cambio
gh pr create      # o abrirlo desde la web de GitHub
```

La documentación del proyecto se escribe **en español**, expandiendo las siglas la primera vez que
aparecen. Cada afirmación sobre el torno se apoya en algo verificado —una foto, una medición, una
prueba de continuidad—, no en lo que "suele ser".

## Licencia

[MIT](LICENSE). El material es de uso libre citando al laboratorio.
