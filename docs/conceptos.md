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

Un paso (principalmente conocido como step) es la unidad minima de movimiento rotacional de un motor paso a paso (en un motor paso a paso comun, por ejemplo, un paso equivale a 1.8°). Para dar un paso, se requiere de un pulso, que en este contexto, es una señal digital de corta duracion que da un salto de 0V a 5V y regresa a 0V.

Un microstepping es una tecnica que permite obtener movimientos rotacionales por debajo del paso o step. este puede llegar a ser de 1/4, 1/8, 1/16, 1/32, 1/64, etc. Por ejemplo, un microstepping de 1/4 requiere de 4 pulsos para dar un paso completo. Esta tecnica permite obtener un movimiento mucho mas suave y mayor resolucion en el posicionamiento. sin embargo, esto tambien hace que se requiera mas torque para superar la incercia del motor y el microcontrolador deberá ser capaz de entregar tales pulsos a una frecuencia mucho mayor.

El husillo es el elemento fundamental que convierte el movimiento rotacional del motor en un desplazamiento lineal sobre el equipamiento. El paso del husillo es la distancia que recorre el eje cuando el motor da una vuelta completa. Por ejemplo, si el husillo tiene un paso de 5 mm y el motor da media vuelta, el eje se desplazara 2.5 mm.


## 2. Lazo abierto

¿Qué quiere decir que el control sea de lazo abierto? Si el motor pierde pasos, ¿quién se entera y
cuándo? ¿Qué cambia con un servo con encoder?

Que el control sea de lazo abierto quiere decir que no existe ninguna realimentacion, o en otras palabras, no se realiza ningun tipo de control sobre la salida. De esta forma, si el motor pierde pasos no existe ningun mecanismo que le avise al sistema de control que se ha producido un error en la salida y por ende, no podra corregirlo.

Cuando se utiliza un servomotor con un encoder, el sistema de control conoce la posicion real del motor a traves del encoder, que es un sensor que mide o detecta la posicion del motor y envia esta informacion al microcontrolador. esto hace que el sistema sepa si el motor, al compararlo con el valor de referencia, ha perdido pasos y pueda corregirlo. En este caso, el control deja de ser de lazo abierto y pasa a ser de lazo cerrado.


## 3. Rampas de aceleración

¿Por qué cae el torque de un motor paso a paso al subir la velocidad? ¿Por qué arrancar de golpe a
alta frecuencia hace perder pasos? ¿Qué forma tiene una rampa y cómo se calcula el tiempo entre
pulsos en cada punto (Austin)?

Es el concepto más importante del proyecto.

El control del sistema se realiza a traves de las rampas de aceleracion ya que se pretende manejar el torque del motor. Para lograr esto, se manipula el intervalo entre pulsos para realizar los pasos correspondientes segun el proceso a desarrollar. sin embargo, este control debe realizarse de forma controlada por las siguientes razones:

- el torque del motor decae cuando sube la velocidad debido a la inductancia de las bobinas: la corriente que pasa a traves de las bobinas no llegan a su valor maximo apenas aumenta la velocidad ya que la corriente sigue la curva exponencial indicada por la constante de tiempo tau. por lo tanto, existe una demora entre el instante que aumenta la corriente y el campo magnetico que se establece en el estator. si este cambio es muy rapido, el torque jamas llegara a su valor final y por ende, el torque se vera reducido permamentemente

- La presencia de la fuerza contraelectromotriz: un aumento en la velocidad provoca un aumento de la fuerza contraelectromotriz, que se opone al movimiento del rotor, reduciendo el torque.

- un tren de pulsos que se traduzca en un cambio abrupto de velocidad haria que el motor se pierda pasos: para lograr el movimiento, primero se debe vencer al torque de inercia. un cambio brusco en la velocidad no permitiria que se desarrolle el torque necesario para vencer esa inercia, provocando que el motor se detenga y se pierdan pasos. esto se mitiga parcialmente con un sistema de control de lazo cerrado, ya que el sistema seria capaz de detectar ese error y corregirlo. sin embargo, esto hace que se tenga que enviar varios pulsos mas, que se traduce en mas corriente y, si no se tiene cuidado, en mayores corrientes nominales de trabajo.

Austin describe que para obtener una rampa de aceleracion adecuada en tiempo real sin necesidad de resolver operaciones costosas computacionalmente, hay que reducir el tiempo entre pulsos de forma gradual. Esto se veria de la siguiente manera (para desacelerar, hay que aumentar gradualmente el tiempo entre pulsos):

![imagen](./img/rampa_aceleracion.png)

para lograr esto, se asume una aceleracion constante durante el proceso de arranque (y de parada tambien).

Matematicamente, esto ultimo se puede expresar como: $$C_n = C_{n-1} - \frac{2 \cdot C_{n-1}}{4 n+1}$$ Donde Cn es el tiempo entre pulsos actuales y Cn-1 es el tiempo entre pulsos anteriores. 

Austin advierte que, en el arranque, el primer pulso se debe calcular a partir de la siguiente ecuacion: $$C_0 = f \sqrt{\frac{2\alpha}{\omega'}}$$

Pero es bastante impreciso, por lo que se debe multiplicar por un factor de correccion.


$$C_0 = 0,676 \cdot f \cdot \sqrt{\frac{2\alpha}{\omega'}}$$

Para desacelerar, se le asigna valores negativos a la posicion $n_i=i-n$. de modo que ahora:

$$C_i = C_{i-1} + \frac{2 \cdot C_{i-1}}{4 (i-m)+1}$$

Esto ultimo se traduce en aumentar el intervalo entre pulsos, lo que provoca que el motor reduzca su velocidad de forma gradual.

## 4. Pulsos por timer

¿Por qué los pulsos salen de una interrupción de timer de alta prioridad y no del loop principal?
¿Qué pasaría con la posición y la velocidad si salieran del loop?

Los pulsos deben salir de una interrupcion de timer por que si salieran del loop principal, la velocidad del motor estaria sujeta a la complejidad de dicho loop principal. si en el mismo hay instrucciones con costos computacionales altos, la velocidad de salida de los pulsos dependera del procesamiento requerido en ese entonces y los pulsos de salida tendran un patron irregular. 

Cuando se trabaja con interrupciones de alta prioridad, apenas se cumple la condicion de interrupcion, el microcontrolador deja de lado lo que estaba haciendo y ejecuta el servicio de interrupcion. Al ser de alta prioridad, siempre se va a ejecutar sin importar que es lo que este haciendo. de esta forma el ritmo de salida de los pulsos seran constantes. 

## 5. Coordinación de ejes

¿Cómo se logra que dos motores lleguen juntos al final de una diagonal? Explicar la idea del
algoritmo de Bresenham aplicado a los pasos.

*(completar)*

## 6. Roscado

¿Por qué el avance tiene que ser RPM × paso de rosca? ¿Por qué cada pasada tiene que empezar en la
misma posición angular del husillo, y qué pasa si no? ¿Qué hace falta en el husillo para lograrlo?
¿Por qué esto es lo único que distingue a un torno de una fresadora en el control?

*(completar)*
