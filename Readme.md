# 🖥️ Custom 64-Bit CPU Architecture

Diseño e implementación de un procesador de **64 bits** con arquitectura personalizada (ISA), desarrollado y simulado íntegramente en **NI Multisim**, acompañado de su propia cadena de herramientas (*toolchain*) en Python para ensamblar código.

> 🚧 **Estado del proyecto:** En desarrollo activo (Fase 1: Diseño de primitivas y ALU de 64 bits).

---

## 🎯 Objetivos del proyecto

* Comprender y construir a bajo nivel el funcionamiento interno de un computador: desde puertas lógicas universales (`NAND`) hasta el ciclo completo de instrucción (*Fetch - Decode - Execute*).
* Implementar una ruta de datos (*Datapath*) modular de 64 bits y su correspondiente Unidad de Control.
* Desarrollar un ensamblador propio en Python capaz de traducir código ensamblador legible a código máquina binario/hexadecimal ejecutable por el hardware simulado.

---

## 🏗️ Arquitectura General (Especificaciones)

| Característica | Especificación |
| :--- | :--- |
| **Ancho de palabra / Buses** | 64 bits |
| **Arquitectura** | RISC personalizada (Load/Store) |
| **Registros de propósito general** | 16 registros de 64 bits (`R0` - `R15`) |
| **ALU** | Operaciones aritméticas, lógicas, desplazamientos y *Flags* (`Z`, `C`, `N`, `V`) |
| **Simulador Hardware** | NI Multisim |
| **Toolchain / Ensamblador** | Python 3 |

---

## 📂 Estructura del Repositorio

```text
├── docs/               # Especificación de la ISA, diagramas de bloques y capturas
├── hardware/           # Circuitos en Multisim (primitivas, bloques 64-bit y CPU completa)
├── toolchain/          # Ensamblador y desensamblador escritos en Python
└── programs/           # Código fuente (.asm) y binarios (.hex) de prueba