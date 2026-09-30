# Mediciones

Resultados de las pruebas, etapa por etapa. Cada tabla dice **con qué** se midió (instrumento,
método) y **en qué condiciones** (tensión de la fuente, corriente del driver, microstepping, carga
sobre el motor): un número sin sus condiciones no se puede repetir ni comparar.

## Etapa 2 — Frecuencia fija

Condiciones: *(completar)*

| Frecuencia comandada (Hz) | Frecuencia medida (Hz) | Velocidad medida (RPM) | Cómo se midió | Observaciones |
|---------------------------|------------------------|------------------------|---------------|---------------|
| | | | | |

## Etapa 3 — Con y sin rampa

Condiciones: *(completar)*

Cómo se detecta la pérdida de pasos: *(completar — por ejemplo, marca en el eje y conteo de vueltas
tras N pulsos, o ir y volver al mismo punto)*.

| Frecuencia final (Hz) | Sin rampa: ¿pierde pasos? | Con rampa: ¿pierde pasos? | Aceleración usada | Observaciones |
|-----------------------|---------------------------|---------------------------|-------------------|---------------|
| | | | | |

## Etapa 4 — Precisión del movimiento con grblHAL

Configuración: pasos/mm = *(completar)*, aceleración = *(completar)*, velocidad máxima =
*(completar)*. Instrumento: *(reloj comparador / calibre)*.

| Comando | Desplazamiento esperado (mm) | Medido (mm) | Error (mm) |
|---------|------------------------------|-------------|------------|
| `G1 X10 F200` | 10 | | |

## Etapa 5 — Repetibilidad del homing

| Intento | Posición tras el homing (lectura del comparador) |
|---------|--------------------------------------------------|
| 1 | |

## Etapa 6 — Husillo

Tolerancia aceptada: *(completar y justificar)*.

| RPM comandada | RPM medida | Instrumento | Diferencia (%) |
|---------------|------------|-------------|----------------|
| | | | |
