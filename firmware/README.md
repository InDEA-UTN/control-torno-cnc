# firmware/

Dos cosas distintas, en dos carpetas:

| Ruta | Etapa | Contenido |
|------|-------|-----------|
| `pruebas/` | 2 y 3 | Código propio y mínimo: pulsos por timer a frecuencia fija, después con rampa de aceleración. No es el firmware final; es para entender qué hace grblHAL por dentro. |
| `grblhal/` | 4 en adelante | La configuración de grblHAL para este torno: plataforma, mapa de pines, pasos/mm, aceleraciones, velocidades máximas. **No se copia el código de grblHAL acá**: se referencia la versión usada (commit o release) y se versiona solo lo propio. |

Todavía vacío: depende de qué microcontrolador se elija (Teensy 4.1 o STM32 Nucleo).

Cuando haya código, este README pasa a explicar cómo compilarlo y cargarlo.
