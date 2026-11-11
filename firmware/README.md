# firmware/

| Ruta / Archivo | Etapa | Contenido |
| :--- | :--- | :--- |
| `firmware_extraido.hex` | 1 | Respaldo intacto de memoria Flash extraída del Arduino ATmega328P original (Grbl v0.8c original a 9600 baudios). |
| `firmware_eeprom.hex` | 1 | Respaldo intacto de memoria EEPROM del microcontrolador original con la calibración activa. |
| `grbl_v08c_torno/` | 1 | Código fuente completo de Grbl v0.8c adaptado y configurado específicamente para el torno y el conexionado soldado. |
| `grbl_v08c_remapeado.hex` | 1 | Binario compilado listo para flashear en el Arduino Nano con pines soldados adaptados (Husillo en D3, Límite X en D10). |
| `pruebas/` | 2 y 3 | Código propio y mínimo: pulsos por timer a frecuencia fija, después con rampa de aceleración. No es el firmware final; es para entender qué hace grblHAL por dentro. |
| grblhal/ | 4 en adelante | La configuración de grblHAL para este torno: plataforma, mapa de pines, pasos/mm, aceleraciones, velocidades máximas. **No se copia el código de grblHAL acá**: se referencia la versión usada (commit o release) y se versiona solo lo propio. |

Los detalles del análisis de ingeniería inversa de las imágenes extraídas se encuentran documentados en [\../docs/caracterizacion.md\](../docs/caracterizacion.md).

Las carpetas de desarrollo (\pruebas/\ y \grblhal/\) se implementarán al seleccionar el microcontrolador objetivo (Teensy 4.1 o STM32 Nucleo).
