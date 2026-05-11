[![Review Assignment Due Date](https://classroom.github.com/assets/deadline-readme-button-22041afd0340ce965d47ae6ef1cefeee28c7c493a6346c4f15d667ab976d596c.svg)](https://classroom.github.com/a/rb0M7Pn8)
[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=23817424&assignment_repo_type=AssignmentRepo)
# Lab07: Visualización en LCD 16x2 usando módulo I²C con microcontrolador PIC


## Integrantes

## Marco Teorico

El protocolo I²C (Inter-Integrated Circuit) es un sistema de comunicación serial de dos hilos que utiliza una línea de datos (SDA) y una de reloj (SCL). Se clasifica como una comunicación half-duplex, lo que significa que solo un dispositivo puede enviar datos por el bus en un momento determinado, a diferencia del protocolo SPI que es full-duplex.

Para el control de la pantalla LCD 16x2 en este laboratorio, se destacan los siguientes puntos:

- Módulo MSSP: El microcontrolador PIC18F45K22 utiliza su módulo MSSP (Master Synchronous Serial Port) configurado en modo maestro para gestionar la generación de reloj, condiciones de inicio/parada y la transmisión de paquetes de datos.

- Expansor PCF8574: Se emplea este adaptador para reducir el uso de pines del microcontrolador de 6-8 pines (modo paralelo) a solo 2 pines (SDA y SCL).

- Direccionamiento: El módulo posee una dirección base de 7 bits (0x27). Al desplazarla para incluir el bit de escritura (0), se obtiene la dirección efectiva de 0x4E utilizada en el código.

- Memoria CGRAM: Para mostrar caracteres especiales, como los niveles de batería, se utiliza la memoria CGRAM de la LCD. Esta permite almacenar hasta 8 caracteres personalizados, definidos como matrices de 5x8 píxeles.

- Gestión de Datos: La comunicación se realiza mediante el envío de la dirección del dispositivo, seguida de bytes de datos que incluyen bits de control (RS, RW, E) y los nibbles de información para la pantalla.


## Documentación
En esta sección debes mencionar que se utilizó el módulo MSSP del PIC18F45K22 configurado como maestro I²C. La conexión se realizó mediante los pines RC3 (SCL) y RC4 (SDA) hacia el expansor PCF8574. El sistema permite controlar una pantalla de 32 caracteres totales (16 columnas x 2 filas) usando solo dos hilos de datos.
## Diagramas

<img width="780" height="510" alt="image" src="https://github.com/user-attachments/assets/1843013f-3108-447d-9a32-cc08a6818f22" />

## Evidencias de implementación
### Parte 1: Visualizacion del texto estatico

<img width="720" height="1280" alt="image" src="https://github.com/user-attachments/assets/e014c621-7668-445f-99a7-f4887319cc6d" />

### Parte 2 con enlace del laboratorio 4 ( Caracteres especiales y desplazamiento)

<img width="899" height="1599" alt="image" src="https://github.com/user-attachments/assets/ab157bc5-4bfd-4f52-8a68-82e529a69e83" />

## Preguntas

1. ¿Por qué I²C se clasifica como half-duplex mientras que SPI es full-duplex? ¿Qué implicación práctica tiene esa diferencia para el control de una LCD?.

- Half-duplex (I²C): Solo un dispositivo puede enviar datos por el bus en un momento dado, utilizando las mismas líneas para transmitir y recibir.

- Full-duplex (SPI): Permite enviar y recibir datos simultáneamente gracias a que posee líneas separadas (SDI y SDO).

- Implicación práctica: Para una LCD, la diferencia es mínima en cuanto a visualización, ya que usualmente solo escribimos datos hacia ella (no leemos de vuelta a alta velocidad). Sin embargo, I²C ahorra pines sacrificando la velocidad de transferencia simultánea que ofrece SPI.

2. En I2C_init() se asigna SSPCON1 = 0x28. Desglose ese valor bit a bit e identifique qué modo de operación del MSSP se está seleccionando y por qué se elige ese valor.

- El valor 0x28 en binario es 0010 1000.

- Bit 5 (SSPEN = 1): Habilita el puerto serie y configura los pines SDA y SCL.

- Bits 3-0 (SSPM = 1000): Selecciona el modo I²C Maestro, donde el reloj se genera por hardware basándose en el registro SSPADD.

- Se elige este valor porque el PIC debe actuar como el controlador principal que inicia y detiene las comunicaciones en el bus.
  
3. Las funciones I2C_start(), I2C_stop() e I2C_write() comparten el mismo patrón: activar un bit de control y luego esperar con while(!PIR1bits.SSPIF). ¿Qué representa la bandera SSPIF y por qué se limpia después de cada operación?.

- SSPIF (Master Synchronous Serial Port Interrupt Flag): Es una bandera de interrupción que se pone en 1 cuando el módulo MSSP completa una operación (como enviar un byte o una condición de Start).

- Limpieza: Se debe limpiar por software (SSPIF = 0) para que el programa pueda detectar correctamente la finalización de la siguiente operación.
  
4. El fuse PBADEN = OFF está presente en la configuración. ¿Qué efecto tendría dejarlo en ON sobre los pines del puerto B, y por qué podría causar problemas si se usan esos pines como salidas digitales?.

- Efecto: Si PBADEN = ON, los pines del Puerto B (RB0-RB4) se configuran como entradas analógicas por defecto al reiniciar el microcontrolador.

- Problema: Si intentas usarlos como salidas digitales para controlar periféricos, el registro de lectura del puerto siempre leerá '0' (modo analógico), impidiendo que la lógica digital funcione correctamente o causando conflictos de lectura/escritura.

5. Compare el control de la LCD en modo paralelo (lab04) con el modo I²C de este laboratorio. Mencione ventajas y desventajas de cada enfoque en términos de: cantidad de pines usados, velocidad de actualización y complejidad del código.

| Característica | Modo Paralelo (Lab04) | Modo I²C (Lab07) |
| :--- | :---: | :---: |
| **Pines utilizados** | 6 a 10 pines (D4-D7, RS, E, etc.) | Solo 2 pines (SDA y SCL) |
| **Protocolo** | Comunicación directa por bus de datos | Protocolo serie sincrónico (MSSP) |
| **Velocidad de refresco** | Muy alta (latencia mínima) | Menor (depende del Baud Rate del I2C) |
| **Complejidad de hardware** | Cableado denso; mayor probabilidad de error | Conexión simple; bus compartido |
| **Complejidad de código** | Baja (manipulación de puertos) | Media (requiere manejo de Start/Stop/Ack) |
| **Escalabilidad** | Difícil (requiere más pines por pantalla) | Alta (múltiples LCDs en el mismo bus) |

6. El bus I²C permite conectar múltiples esclavos con solo dos hilos. Si se quisiera agregar un segundo módulo PCF8574 al mismo bus (por ejemplo, para controlar un segundo LCD), ¿qué cambio mínimo sería necesario en el hardware y en el código?

- Hardware: Se debe cambiar la dirección física del segundo PCF8574 mediante sus pines de dirección (A0, A1, A2), usualmente puenteando unos pads en el módulo.

- Código: Se debe definir una nueva dirección constante (ej. #define ADDRESS_LCD2 0x4C) y llamar a las funciones de escritura usando esa nueva dirección para diferenciar a qué pantalla se le envía la información.

## Conclusiones
- La implementación del protocolo I²C mediante el módulo MSSP reduce drásticamente el uso de recursos de hardware (pines) del microcontrolador.
  
- El uso de la memoria CGRAM permite expandir las capacidades de la interfaz de usuario mediante caracteres personalizados como iconos de batería.
  
- La modularización del código en archivos .c y .h facilita el mantenimiento y la portabilidad del firmware entre diferentes proyectos.

## Referencias
