# Becerra-post1-u3

## Parte 1: Exploración con DEBUG en DOSBox

### Parte A — Configuración del Entorno DOSBox

#### Configuración e Inicio
Se procedió a crear la carpeta de trabajo en el sistema anfitrión y a realizar el montaje del directorio local dentro de DOSBox para establecer la unidad virtual `C:`. 
Posteriormente, se creó la subcarpeta de trabajo específica del laboratorio y se inició la herramienta de depuración `DEBUG`.

Comandos ejecutados en el prompt de DOS:
```cmd
Z:\> MOUNT C C:\DOSWork
C:\> MD LAB3POST1
C:\> CD LAB3POST1
C:\LAB3POST1> DEBUG
```

### Parte B — Inspección de Registros con el Comando R
### Inspección Inicial del Procesador
Al invocar el depurador sin parámetros mediante el comando R, se obtuvo la lectura del estado inicial del procesador x86:
-R
AX=0000  BX=0000  CX=0000  DX=0000  SP=FFFE  BP=0000  SI=0000  DI=0000
DS=1357  ES=1357  SS=1357  CS=1357  IP=0100   NV UP EI PL NZ NA PO NC
1357:0100 CD20          INT 20
Observaciones Técnicas:
Registros de propósito general: Los registros AX, BX, CX y DX inician en 0000h.

Puntero de pila: El registro SP apunta a FFFEh, representando la dirección más alta disponible de la pila inicial.

Registros de segmento: Los registros DS, ES, SS y CS contienen el mismo valor de segmento (1357h), el cual corresponde al segmento del PSP (Program Segment Prefix) asignado por el sistema operativo DOS.

Puntero de instrucción: El registro IP inicia en 0100h, la primera dirección lógica ejecutable inmediatamente después del bloque del PSP.

Se utilizó el comando R AX para modificar el valor del registro acumulador, asignándole el valor hexadecimal 1234 (4660 en decimal)

### Parte C — Volcado de Memoria con D y Relleno con F

#### Paso 6: Relleno de Bloque de Memoria con Patrón
Se utilizó el comando `F` (Fill) para inicializar un bloque de 64 bytes (`40h`) a partir de la dirección `DS:0200` con la secuencia cíclica de bytes `AB CD EF`:

```text
-F 200 L40 AB CD EF
```

Para verificar la escritura en memoria, se ejecutó el comando D (Dump) sobre el mismo rango de 64 bytes:

### Paso 7: Volcado del Bloque de Memoria
-D 200 L40

La columna lateral derecha muestra únicamente puntos (.) debido a que los valores hexadecimales ABh, CDh y EFh se encuentran fuera del rango de caracteres ASCII imprimibles (el cual abarca de 20h a 7Eh).

Se realizó la inspección de los primeros 32 bytes (20h) del Program Segment Prefix (PSP) desde la dirección relativa 0000h:

### Paso 8: Exploración del Área del PSP
-D 0 L20
Análisis de la instrucción de terminación:
Los primeros dos bytes corresponden a CD 20, que codifican la instrucción INT 20h. Esta instrucción es colocada por el sistema operativo DOS al inicio del PSP como un mecanismo de salida de emergencia. Si una rutina finaliza y retorna a la dirección 0000h, se ejecuta INT 20h para terminar la ejecución de forma controlada.

### Parte E — Modificación de Memoria con E y Direccionamiento Directo

#### Paso 11: Modificación de Bytes Puntuales con E
Se limpió un bloque de 16 bytes (`10h`) a partir de la dirección `DS:0300` inicializándolo en cero mediante el comando `F`. Posteriormente, se utilizó el comando `E` (Enter) para escribir los bytes específicos `78h` y `56h`

Observación:
El segundo volcado con D confirma que el comando E alteró de forma exclusiva los dos primeros bytes en 1357:0300 y 1357:0301, conservando intactos los 14 bytes restantes. Leídos en formato little-endian, estos bytes representan la palabra de 16 bits 5678h.

### Paso 12: Decisión Técnica — Verificación No Destructiva de una Escritura en Memoria
Para verificar la correcta modificación de la memoria sin alterar el estado actual del procesador ni del programa, el comando adecuado es D (Dump).

A diferencia de D, invocar el comando E sin la lista de bytes inicia un modo interactivo en el depurador que muestra el byte actual y espera la entrada del usuario por teclado. Este comportamiento representa un riesgo operativo, ya que cualquier pulsación accidental de teclas o el uso de la tecla Enter o la barra espaciadora podría sobrescribir por error el contenido de la memoria antes de completar la verificación.

Por otro lado, el comando F (Fill) tampoco es adecuado como mecanismo de verificación debido a que es una instrucción de escritura destructiva; su propósito es sobreescribir masivamente un rango de memoria con un patrón determinado, por lo que utilizarlo para inspeccionar terminaría borrando o modificando los datos previamente almacenados.

Finalmente, la razón por la cual D es el único candidato completamente idóneo es que se trata de un comando de solo lectura. A diferencia de R (que permite alterar registros), E (que modifica bytes específicos) y F (que modifica bloques enteros), D únicamente lee la memoria principal y la traslada a la pantalla en formato hexadecimal y ASCII, garantizando que el estado de los registros, las banderas y la memoria permanezcan intactos durante la inspección.
