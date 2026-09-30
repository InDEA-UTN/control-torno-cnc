# Conceptos clave

**Entregable de la etapa 0.** Cada concepto se explica **con palabras propias**, no copiando la
fuente: si no se puede explicar sin el texto al lado, todavía no se entendió. Donde ayude, un
ejemplo con números o un dibujo. Al final de cada uno, de dónde se sacó.

Las preguntas debajo de cada título son una guía de qué tiene que quedar respondido; se borran
cuando la sección está escrita.

## 1. Pasos por milímetro

> pasos/mm = (pasos por vuelta × microstepping) / paso del husillo en mm

¿Qué es cada término? ¿Qué es el microstepping y qué se gana y qué se pierde con él? Un ejemplo
numérico con valores razonables para un torno chico.

*(completar)*

## 2. Lazo abierto

¿Qué quiere decir que el control sea de lazo abierto? Si el motor pierde pasos, ¿quién se entera y
cuándo? ¿Qué cambia con un servo con encoder?

*(completar)*

## 3. Rampas de aceleración

¿Por qué cae el torque de un motor paso a paso al subir la velocidad? ¿Por qué arrancar de golpe a
alta frecuencia hace perder pasos? ¿Qué forma tiene una rampa y cómo se calcula el tiempo entre
pulsos en cada punto (Austin)?

Es el concepto más importante del proyecto.

*(completar)*

## 4. Pulsos por timer

¿Por qué los pulsos salen de una interrupción de timer de alta prioridad y no del loop principal?
¿Qué pasaría con la posición y la velocidad si salieran del loop?

*(completar)*

## 5. Coordinación de ejes

¿Cómo se logra que dos motores lleguen juntos al final de una diagonal? Explicar la idea del
algoritmo de Bresenham aplicado a los pasos.

*(completar)*

## 6. Roscado

¿Por qué el avance tiene que ser RPM × paso de rosca? ¿Por qué cada pasada tiene que empezar en la
misma posición angular del husillo, y qué pasa si no? ¿Qué hace falta en el husillo para lograrlo?
¿Por qué esto es lo único que distingue a un torno de una fresadora en el control?

*(completar)*
