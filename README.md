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

Se utilizó el comando R AX para modificar el valor del registro acumulador, asignándole el valor hexadecimal 1234 (4660 en decimal):
