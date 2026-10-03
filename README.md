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

## Parte 2: Ejecución Paso a Paso y Análisis de Traza

### Parte A — Programa de Suma con Traza Completa

#### Paso 1 y 2: Preparación del Entorno y Ensamblado del Programa
Se configuró la subcarpeta de trabajo `LAB3POST2` en la unidad virtual `C:` de DOSBox y se procedió a ensamblar en la dirección `0100h` un programa que realiza la suma de tres operandos almacenados en registros de propósito general.

Paso,Instrucción Ejecutada,AX,BX,CX,IP Sig.,ZF (Zero),CF (Carry),SF (Sign)
Inicio,(Estado Inicial),0000,0000,0000,0100,NZ (0),NC (0),PL (0)
1,"MOV AX, 000A",000A,0000,0000,0103,NZ (0),NC (0),PL (0)
2,"MOV BX, 0005",000A,0005,0000,0106,NZ (0),NC (0),PL (0)
3,"MOV CX, 0003",000A,0005,0003,0109,NZ (0),NC (0),PL (0)
4,"ADD AX, BX",000F,0005,0003,010B,NZ (0),NC (0),PL (0)
5,"ADD AX, CX",0012,0005,0003,010D,NZ (0),NC (0),PL (0)

### Parte B — Programa con Bucle usando LOOP y CX
#### Paso 5 y 6: Ensamblado y Verificación del Bucle
Se procedió a ensamblar en la dirección `0100h` un programa que implementa un bucle mediante la instrucción `LOOP`, utilizando el registro `CX` como contador de iteraciones para realizar la suma repetitiva del valor `0002h` cuatro veces consecutivas:

```text
-A 100
1357:0100 MOV AX, 0000
1357:0103 MOV CX, 0004
1357:0106 ADD AX, 0002
1357:0109 LOOP 0106
1357:010B INT 20
```

Paso / Iteración,Instrucción Ejecutada,AX después,CX después,IP Sig.,¿LOOP salta?
Inicio,"MOV AX, 0000",0000,0004,0103,N/A
Inicio,"MOV CX, 0004",0000,0004,0106,N/A
Iteración 1,"ADD AX, 0002",0002,0004,0109,N/A
Iteración 1,LOOP 0106,0002,0003,0106,Sí (CX=0)
Iteración 2,"ADD AX, 0002",0004,0003,0109,N/A
Iteración 2,LOOP 0106,0004,0002,0106,Sí (CX=0)
Iteración 3,"ADD AX, 0002",0006,0002,0109,N/A
Iteración 3,LOOP 0106,0006,0001,0106,Sí (CX=0)
Iteración 4,"ADD AX, 0002",0008,0001,0109,N/A
Iteración 4,LOOP 0106,0008,0000,010B,No (CX=0)
Final,INT 20,0008,0000,--,Program terminated

### Parte C — Análisis del Código Máquina con D

#### Paso 8: Comparación de Código Máquina y Ensamblador
Se utilizó el comando `D` sobre el segmento de código del programa del bucle (`CS:100 L0C`) para inspeccionar la codificación hexadecimal directa de las instrucciones:

```text
-D CS:100 L0C
1357:0100  B8 00 00 B9 04 00 05 02-00 E2 FB CD 20
```

B8 00 00  
MOV AX, 0000 (3 bytes)B9 04 00 
MOV CX, 0004 (3 bytes)05 02 00 
ADD AX, 0002 (3 bytes)E2 FB 
LOOP 0106 (2 bytes)CD 20 
INT 20 (2 bytes)

El programa completo ocupa un total de 13 bytes en memoria. Se evidencia cómo instrucciones complejas como LOOP logran empaquetar en tan solo 2 bytes operaciones que de otro modo requerirían múltiples instrucciones.

### Parte D — Bucle Equivalente con DEC/JNZ y Comparación de Costo de Instrucciones
### Paso 9: Ensamblado y Verificación del Bucle Equivalente
Se ensambló en la dirección 0200h una versión equivalente del bucle que sustituye la instrucción LOOP por la combinación manual de decremento de registro DEC CX y salto condicional JNZ

### Paso 10: Decisión Técnica — Selección de Mecanismo de Control de Bucle (LOOP vs. DEC/JNZ)
Para un bucle contador simple donde el registro CX se utiliza únicamente como control de iteraciones, se recomienda el uso de la instrucción LOOP. Esta recomendación se justifica en la optimización del código máquina: LOOP requiere únicamente 2 bytes (E2 FB), mientras que la alternativa mediante DEC CX y JNZ suma 3 bytes (49 75 FA). Adicionalmente, LOOP requiere que la unidad de control del procesador realice un solo ciclo de búsqueda (fetch) por iteración en lugar de dos, reduciendo el tráfico en el bus de instrucciones.

Sin embargo, el mecanismo DEC/JNZ es preferible en escenarios más complejos. Por ejemplo, si dentro del cuerpo del bucle se requiere reutilizar el registro CX para operaciones aritméticas u otros fines, el conteo no dependería exclusivamente de CX (pudiéndose usar otro registro como BX con DEC BX). También es indispensable cuando la condición de salida no es simplemente un conteo regresivo a cero, sino el resultado de una comparación lógica explícita mediante la instrucción CMP evaluando diferentes banderas (como Carry Flag o Sign Flag).

Para verificar cuál versión ocupa menos bytes sin ejecutar ninguna instrucción, el estudiante utiliza el comando U (Unassemble). Al inspeccionar la columna de offsets en la salida del desensamblado, se calcula la diferencia entre la dirección de la primera instrucción del bucle y la dirección inmediatamente posterior. En LOOP la diferencia de direcciones confirma 2 bytes de ocupación frente a los 3 bytes visibles entre DEC y JNZ.

### Parte E — Bucle Equivalente con DEC/JNZ y Comando G

#### Paso 11: Traza del Bucle DEC/JNZ y Conteo Comparativo de Instrucciones
Se realizó la traza instrucción a instrucción mediante el comando `T` para evaluar la ejecución del bucle implementado con `DEC CX` y `JNZ 0206`.

#### Tabla de Traza del Bucle DEC/JNZ:

| Paso | Instrucción Ejecutada | AX después | CX después | IP Sig. | ¿JNZ salta? |
| :---: | :--- | :---: | :---: | :---: | :---: |
| **Inicio** | `MOV AX, 0000` | `0000` | `0004` | `0203` | N/A |
| **Inicio** | `MOV CX, 0004` | `0000` | `0004` | `0206` | N/A |
| **Iteración 1** | `ADD AX, 0002` | `0002` | `0004` | `0209` | N/A |
| **Iteración 1** | `DEC CX` | `0002` | `0003` | `020A` | N/A |
| **Iteración 1** | `JNZ 0206` | `0002` | `0003` | `0206` | Sí ($ZF = 0$) |
| **Iteración 2** | `ADD AX, 0002` | `0004` | `0003` | `0209` | N/A |
| **Iteración 2** | `DEC CX` | `0004` | `0002` | `020A` | N/A |
| **Iteración 2** | `JNZ 0206` | `0004` | `0002` | `0206` | Sí ($ZF = 0$) |
| **Iteración 3** | `ADD AX, 0002` | `0006` | `0002` | `0209` | N/A |
| **Iteración 3** | `DEC CX` | `0006` | `0001` | `020A` | N/A |
| **Iteración 3** | `JNZ 0206` | `0006` | `0001` | `0206` | Sí ($ZF = 0$) |
| **Iteración 4** | `ADD AX, 0002` | `0008` | `0001` | `0209` | N/A |
| **Iteración 4** | `DEC CX` | `0008` | `0000` | `020A` | N/A |
| **Iteración 4** | `JNZ 0206` | `0008` | `0000` | `020C` | **No** ($ZF = 1$) |
| **Final** | `INT 20` | `0008` | `0000` | `--` | Program terminated |

#### Conteo Comparado de Instrucciones Ejecutadas:
* **Versión con `LOOP`:** Ejecuta un total de **11 instrucciones** (2 de inicialización + 4 × 2 del bloque interno + 1 de terminación).
* **Versión con `DEC/JNZ`:** Ejecuta un total de **15 instrucciones** (2 de inicialización + 4 × 3 del bloque interno + 1 de terminación).
* **Conclusión:** El mecanismo `DEC/JNZ` requiere **4 instrucciones adicionales** para completar el mismo número de iteraciones (una instrucción extra por ciclo).

---

#### Paso 12: Decisión Técnica — Comando de Verificación para Bucles de Muchas Iteraciones (T vs. G)

Para bucles de muchas iteraciones (como $CX = 100$), el uso del comando **`G` (Go)** es drásticamente más práctico que `T`, ya que ejecuta el programa a velocidad real del procesador hasta alcanzar la dirección de parada indicada (ej. `G 20C`), ahorrando la ejecución manual de cientos de comandos `T`. La desventaja principal de utilizar `G` es la pérdida total de visibilidad sobre los estados intermedios del programa: no es posible observar la evolución gradual de los registros, el decremento paso a paso del contador ni los cambios temporales en las banderas de condición.

El uso de **`T`** fue indispensable en los Pasos 4, 7 y 11 del laboratorio debido a que el objetivo didáctico era analizar el comportamiento detallado del microprocesador e inspeccionar el estado exacto de los registros tras cada instrucción individual para completar las tablas de traza.

Tras ejecutar `G 20C`, si el estudiante desea verificar únicamente el valor del registro `AX` sin inspeccionar los demás componentes, puede ejecutar el comando específico **`R AX`**, el cual muestra el contenido del acumulador sin alterar ningún dato en memoria.
