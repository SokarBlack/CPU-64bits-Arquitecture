# 📘 Especificación de la Arquitectura del Conjunto de Instrucciones (ISA)

Este documento define la arquitectura de instrucciones implementada en el procesador de 64 bits simulado en NI Multisim.

---

## 1. Visión General de la Arquitectura

* **Tipo de Arquitectura:** RISC (Reduced Instruction Set Computer) con modelo *Load/Store*.
* **Ancho de Datos (Datapath):** 64 bits (todos los registros y buses de datos de la ALU operan sobre palabras de 64 bits en complemento a dos).
* **Ancho de Instrucción:** 32 bits de longitud fija.
* **Banco de Registros:** 16 registros de propósito general de 64 bits (`R0` – `R15`), direccionados mediante campos de 4 bits.
* **Endianness:** Little-Endian / Direccionamiento por palabra de 64 bits (cada incremento de `+1` en el `PC` apunta a la siguiente instrucción de 32 bits en la ROM).

---

## 2. Banco de Registros (`Register File`)

El procesador cuenta con 16 registros internos de 64 bits (`R0` a `R15`). El registro `R0` está cableado permanentemente a tierra (`0x0000000000000000`), por lo que cualquier intento de escritura sobre él es ignorado.

| Registro | Código Binario | Alias / Convención | Descripción |
| :--- | :---: | :---: | :--- |
| **`R0`** | `0000` | `$zero` | Constante cero (`0`) cableada por hardware (solo lectura). |
| **`R1` – `R3`** | `0001` – `0011` | `$v0` – `$v2` | Resultados de evaluación y valores de retorno. |
| **`R4` – `R7`** | `0100` – `0111` | `$a0` – `$a3` | Argumentos y parámetros de entrada. |
| **`R8` – `R13`** | `1000` – `1101` | `$t0` – `$t5` | Registros temporales de propósito general. |
| **`R14`** | `1110` | `$sp` | Puntero de pila (*Stack Pointer*) / Puntero base de memoria. |
| **`R15`** | `1111` | `$ra` | Dirección de retorno o registro auxiliar del sistema. |

---

## 3. Formatos de Instrucción (32 bits)

Todas las instrucciones tienen una longitud fija de **32 bits** y se dividen en tres formatos estándar: **Tipo R** (Registro), **Tipo I** (Inmediato / Memoria / Salto condicional) y **Tipo J** (Salto incondicional).

### 3.1 Formato Tipo R (Operaciones ALU entre registros)
Se utiliza para operaciones aritméticas y lógicas donde los dos operandos y el destino son registros. Todas las instrucciones Tipo R comparten el `Opcode = 000000` y se distinguen mediante el campo `Funct`.

| Bits `[31:26]` (6 bits) | Bits `[25:22]` (4 bits) | Bits `[21:18]` (4 bits) | Bits `[17:14]` (4 bits) | Bits `[13:6]` (8 bits) | Bits `[5:0]` (6 bits) |
| :---: | :---: | :---: | :---: | :---: | :---: |
| **`Opcode`** | **`rs`** | **`rt`** | **`rd`** | **`Reservado`** | **`Funct`** |
| `000000` | Registro fuente 1 | Registro fuente 2 | Registro destino | `00000000` | Código de función ALU |

### 3.2 Formato Tipo I (Inmediatos, Acceso a Memoria y Saltos Condicionales)
Se utiliza para operar con una constante de 18 bits con signo (*Inmediato*), calcular direcciones de memoria (`LD`/`SD`) o realizar saltos relativos (`BEQ`/`BNE`).

| Bits `[31:26]` (6 bits) | Bits `[25:22]` (4 bits) | Bits `[21:18]` (4 bits) | Bits `[17:0]` (18 bits) |
| :---: | :---: | :---: | :---: |
| **`Opcode`** | **`rs`** | **`rt`** | **`Immediate`** |
| Código de operación | Registro fuente / base | Registro destino / fuente | Valor inmediato (extensión de signo a 64 bits) |

### 3.3 Formato Tipo J (Salto Incondicional)
Se utiliza para saltar directamente a una dirección absoluta de instrucción en la ROM.

| Bits `[31:26]` (6 bits) | Bits `[25:0]` (26 bits) |
| :---: | :---: |
| **`Opcode`** | **`Target Address`** |
| `000010` | Dirección absoluta de destino en memoria de instrucciones |

---

## 4. Conjunto de Instrucciones (12 Instrucciones Base)

### 4.1 Tabla Resumen de Opcodes y Códigos de Función

| Mnemónico | Tipo | Opcode `[31:26]` | Funct `[5:0]` | Sintaxis Ensamblador | Operación RTL (64 bits) |
| :--- | :---: | :---: | :---: | :--- | :--- |
| **`ADD`** | R | `000000` | `100000` | `ADD rd, rs, rt` | `R[rd] ← R[rs] + R[rt]` |
| **`SUB`** | R | `000000` | `100010` | `SUB rd, rs, rt` | `R[rd] ← R[rs] - R[rt]` |
| **`AND`** | R | `000000` | `100100` | `AND rd, rs, rt` | `R[rd] ← R[rs] & R[rt]` |
| **`OR`** | R | `000000` | `100101` | `OR rd, rs, rt` | `R[rd] ← R[rs] \| R[rt]` |
| **`XOR`** | R | `000000` | `100110` | `XOR rd, rs, rt` | `R[rd] ← R[rs] ^ R[rt]` |
| **`SLT`** | R | `000000` | `101010` | `SLT rd, rs, rt` | `R[rd] ← (R[rs] < R[rt]) ? 1 : 0` |
| **`ADDI`** | I | `001000` | — | `ADDI rt, rs, imm` | `R[rt] ← R[rs] + SignExt(imm)` |
| **`LD`** | I | `110111` | — | `LD rt, imm(rs)` | `R[rt] ← RAM[R[rs] + SignExt(imm)]` |
| **`SD`** | I | `111111` | — | `SD rt, imm(rs)` | `RAM[R[rs] + SignExt(imm)] ← R[rt]` |
| **`BEQ`** | I | `000100` | — | `BEQ rs, rt, offset` | `if (R[rs] == R[rt]) PC ← PC + 1 + SignExt(offset)` |
| **`BNE`** | I | `000101` | — | `BNE rs, rt, offset` | `if (R[rs] != R[rt]) PC ← PC + 1 + SignExt(offset)` |
| **`J`** | J | `000010` | — | `J target` | `PC ← target` |

---

### 4.2 Descripción Detallada por Instrucción

#### Instrucciones Aritméticas y Lógicas (Tipo R)
* **`ADD rd, rs, rt`**: Suma el contenido de 64 bits de `rs` y `rt` y guarda el resultado en `rd`.
* **`SUB rd, rs, rt`**: Resta el contenido de `rt` al de `rs` (`rs - rt`) y guarda el resultado en `rd`.
* **`AND rd, rs, rt`**: Realiza la operación lógica `AND` bit a bit entre `rs` y `rt`, almacenando el resultado en `rd`.
* **`OR rd, rs, rt`**: Realiza la operación lógica `OR` bit a bit entre `rs` y `rt`, almacenando el resultado en `rd`.
* **`XOR rd, rs, rt`**: Realiza la operación lógica `XOR` bit a bit entre `rs` y `rt`, almacenando el resultado en `rd`.
* **`SLT rd, rs, rt`** *(Set on Less Than)*: Compara ambos valores con signo; si `R[rs] < R[rt]`, escribe `1` en `rd`; en caso contrario, escribe `0`. Fundamental para construir bucles y condiciones mayores/menores junto con `BEQ`/`BNE`.

#### Instrucciones Inmediatas y de Memoria (Tipo I)
* **`ADDI rt, rs, imm`**: Extiende el signo del inmediato de 18 bits a 64 bits, lo suma a `R[rs]` y almacena el resultado en `R[rt]`. *(Nota: Se utiliza `ADDI rt, R0, valor` para cargar constantes iniciales en un registro).*
* **`LD rt, imm(rs)`** *(Load Doubleword)*: Calcula la dirección efectiva sumando `R[rs] + SignExt(imm)` y carga la palabra de **64 bits** almacenada en esa dirección de la memoria RAM dentro del registro `rt`.
* **`SD rt, imm(rs)`** *(Store Doubleword)*: Calcula la dirección efectiva sumando `R[rs] + SignExt(imm)` y escribe el contenido de **64 bits** del registro `rt` en esa posición de la memoria RAM.

#### Instrucciones de Control de Flujo (Tipo I y Tipo J)
* **`BEQ rs, rt, offset`** *(Branch if Equal)*: Compara `R[rs]` y `R[rt]`. Si son iguales (bandera `Zero = 1` en la ALU), salta a la instrucción indicada por `PC + 1 + SignExt(offset)`. Si no son iguales, continúa secuencialmente (`PC + 1`).
* **`BNE rs, rt, offset`** *(Branch if Not Equal)*: Compara `R[rs]` y `R[rt]`. Si son distintos (bandera `Zero = 0` en la ALU), salta a `PC + 1 + SignExt(offset)`.
* **`J target`** *(Jump)*: Salto incondicional. Carga directamente los 26 bits de `target` en el Contador de Programa (`PC`).

---

## 5. Señales de Control de la ALU (`ALUControl`)

La Unidad de Control traduce los campos `Opcode` y `Funct` en una señal interna de **3 bits** (`ALUOp`) que gobierna las operaciones de la ALU de 64 bits:

| `ALUControl` (3 bits) | Operación de la ALU | Instrucciones que la utilizan |
| :---: | :--- | :--- |
| `000` | `AND` bit a bit | `AND` |
| `001` | `OR` bit a bit | `OR` |
| `010` | Suma (`A + B`) | `ADD`, `ADDI`, `LD`, `SD` |
| `011` | `XOR` bit a bit | `XOR` |
| `110` | Resta (`A - B`) | `SUB`, `BEQ`, `BNE` |
| `111` | Comparación (`A < B ? 1 : 0`) | `SLT` |

---

## 6. Ejemplo de Codificación Binaria

A continuación se muestra cómo se traduce un fragmento de código ensamblador a palabras de 32 bits listas para cargar en la memoria de instrucciones (ROM):

```asm
ADDI R1, R0, 5      # R1 = 0 + 5 = 5
ADDI R2, R0, 3      # R2 = 0 + 3 = 3
ADD  R3, R1, R2     # R3 = R1 + R2 = 8
SD   R3, 0(R0)      # RAM[0] = 8