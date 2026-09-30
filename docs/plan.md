# Plan de trabajo

Las etapas son una guía, no un contrato: si el proyecto pide reordenarlas, se reordenan y se anota
acá el porqué. Las duraciones son estimaciones en semanas de trabajo; lo que manda es el **criterio
de éxito**: una etapa se da por cerrada cuando se cumple, no cuando pasa el tiempo.

## Estado actual

**Etapa 0** — preparación.

## Etapas

### Etapa 0 — Preparación *(0,5 semana)*

Leer a David Austin, *Generate stepper-motor speed profiles in real time*, y el manual de un driver
de motor paso a paso típico (A4988, DRV8825 o TB6600). Ver [`referencias.md`](referencias.md).

**Éxito:** [`conceptos.md`](conceptos.md) completo — los seis conceptos explicados con palabras
propias.

### Etapa 1 — Caracterizar el torno *(1–2 semanas)*

Fotografiar todo antes de desconectar nada. Identificar:

- **Motores de los ejes:** ¿paso a paso, o servo con encoder?
- **Drivers:** ¿placas separadas con entradas STEP/DIR/ENABLE, o una placa propietaria?
- **Motor del husillo:** ¿velocidad fija, PWM (modulación por ancho de pulso) o relé? ¿Tiene
  encoder o sensor de índice?
- **Finales de carrera, home y parada de emergencia:** cuáles hay, dónde y cómo están cableados.

Seguir con el tester en modo continuidad cada cable del conector DB25 hasta su destino. Las salidas
típicas del puerto paralelo son los pines 2 a 9 y las entradas típicas 10 a 13 y 15: **verificarlo,
no asumirlo.** Si hay optoacopladores en el medio, mirar el puerto con analizador lógico u
osciloscopio con el equipo en movimiento.

**Éxito:** mapa de pines y esquema completos en [`caracterizacion.md`](caracterizacion.md).

**Riesgo:** si los ejes resultan ser servos con lazo cerrado propietario, o la placa tiene su propia
lógica de movimiento (no es un simple STEP/DIR), **el alcance se replantea** antes de seguir.
Avisar apenas aparezca.

### Etapa 2 — Un motor, un driver, un microcontrolador *(1 semana)*

Pulsos a frecuencia fija generados por un timer del microcontrolador. Un solo motor, fuera del
torno o con el carro lejos de los topes.

**Éxito:** el motor gira a una velocidad controlada **y medida** (no solo comandada).

### Etapa 3 — Rampas de aceleración *(1–2 semanas)*

Implementar una rampa de aceleración y desaceleración según Austin.

**Éxito:** el motor alcanza una velocidad a la que antes, arrancando de golpe, perdía pasos. Las
frecuencias probadas, con y sin rampa, quedan documentadas en [`mediciones.md`](mediciones.md).

Es la etapa donde más se traba la gente: está bien que lleve más de lo previsto.

### Etapa 4 — grblHAL sobre el mismo hardware *(1–2 semanas)*

Cargar grblHAL en el mismo microcontrolador y mover un eje real con G-code. Configurar pasos/mm,
aceleración y velocidad máxima. Leer `stepper.c` y `planner.c` de Grbl y compararlos con lo hecho
en las etapas 2 y 3: qué hace igual, qué hace distinto y por qué.

**Éxito:** `G1 X10 F200` mueve **exactamente 10 mm**, verificado con reloj comparador o calibre.

### Etapa 5 — Dos ejes, homing, finales de carrera y parada de emergencia *(1–2 semanas)*

Placa de interfaz definitiva, con buffers u optoacopladores y entradas protegidas. La parada de
emergencia va **por hardware**, cortando la potencia con contactos normalmente cerrados, nunca solo
por software. Contra el ruido, desde el principio: masas separadas, optoacopladores y cable
blindado.

**Éxito:** homing repetible, y una parada de emergencia que corta la potencia sin intervención del
firmware.

### Etapa 6 — Husillo *(1–3 semanas)*

Control de velocidad del husillo (PWM o relé, según lo que se haya encontrado en la etapa 1) y
lectura de RPM (revoluciones por minuto).

**Éxito:** la RPM comandada y la medida coinciden dentro de una tolerancia documentada.

### Etapa 7 — Roscado *(2–4 semanas, opcional)*

Solo con todo lo anterior estable. Sincronizar el avance del carro con la posición angular del
husillo.

**Éxito:** una rosca de prueba con el paso correcto, verificada con peine de roscas o calibre.
