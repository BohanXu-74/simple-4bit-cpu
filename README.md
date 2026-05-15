# BX-4 — A 4-Bit CPU Series

[![Demo Video](https://img.shields.io/badge/YouTube-Demo%20Video-red?logo=youtube)](https://www.youtube.com/watch?v=geT2ecY6J6I)
[![Hackaday](https://img.shields.io/badge/Hackaday.io-Project%20Page-brightgreen)](https://hackaday.io/project/204999-student-made-ttl-cpu)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

![BX-4.1 Photo](images/cpu.png)

This repository is the home of the **BX-4**, a custom 4-bit CPU designed from scratch starting in 5th grade after reading *Code* by Charles Petzold. BX stands for Bohan Xu. The series currently has three versions: BX-4.1, BX-4.2, and BX-4.3. The first was physically built from 74-series TTL chips and actually runs. The later two are fully simulated in Digital by H. Neemann but have not been built yet.

This project is also the foundation for everything that came after it, including an 8-bit CPU and the [APEX-16](https://github.com/BohanXu-74/APEX-16), a 16-bit pipelined CPU currently in development.

---

## The Versions at a Glance

| Version | Status | Key Improvement |
|---|---|---|
| BX-4.1 | Physically built and working | Original design |
| BX-4.2 | Fully simulated, not built | Fewer cycles per instruction, techniques inspired by 8051 architecture |
| BX-4.3 | Fully simulated, not built | Further refinement of BX-4.2, cleaner ROM format |

---

## Background

I wanted to understand how a processor actually works at the lowest level, not just conceptually but gate by gate. So after reading *Code* by Charles Petzold, I designed and built one. The BX-4 is implemented entirely in 74-series TTL logic chips and is closer to a microcontroller than a true microprocessor. There is no external RAM access. The program lives in ROM simulated by an Arduino. It is slow and it needed a lot of fighting to get running reliably, but it works and it executes real instructions on real hardware.

---

## Architecture Overview

| Property | Value |
|---|---|
| Data width | 4-bit |
| Program counter width | 12-bit |
| Addressable program space | 4096 instruction locations |
| Registers | 4 general purpose (R1, R2, R3, R4) |
| ALU operations | ADD, bitwise complement (invert) |
| Instruction count | 15 instructions |
| Logic family | 74-series TTL |
| Simulation tool | Digital by H. Neemann |

The 12-bit program counter gives the CPU 4096 addressable locations in program memory. All four registers are general purpose with no dedicated accumulator or stack pointer at this level.

---

## Instruction Set

The BX-4 supports 15 instructions. The core ones are:

| Instruction | Description |
|---|---|
| `MOV #num, Rn` | Load an immediate 4-bit value directly into a register |
| `MOV Rn, Rm` | Copy the contents of one register into another |
| `ADD Rn, Rm` | Add two registers, result stored back |
| `CMP Rn, Rm` | Compare two registers and set flags |
| `JMP addr` | Unconditional jump to a 12-bit address |
| `COMPLEMENT Rn` | Bitwise invert a register (NOT operation) |

The ALU handles addition and complement natively. Subtraction works through complement and add, which is the standard two's complement method used in real processor designs too.

---

## How the ROM Simulator Works

Instead of a physical ROM chip, program memory is simulated by an Arduino. On each clock pulse, the Arduino outputs the current instruction to the CPU's data bus and tracks the program counter internally.

When a `JMP` instruction executes, the CPU latches the 12-bit jump target address directly onto 12 of the Arduino's I/O pins. The Arduino reads that address and moves its internal program counter to match, so the next clock pulse outputs the instruction at the new location. You can also write new programs directly into the Arduino's memory without rewiring anything.

---

## Version Details

### BX-4.1
The original design and the only version that has been physically built. It works, though it needed deliberate timing delays added at various points to deal with propagation delay and racing signals. Logic was designed and verified in Digital before being wired up on real hardware.

### BX-4.2
A full redesign informed by techniques from the 8051 architecture. Instructions execute in fewer clock cycles than BX-4.1. This version also includes a custom assembler written by [Bolan Xu](https://github.com/bolanxu) ([@TheBitDude](https://www.youtube.com/@TheBitDude)), along with hand-written assembly and machine code examples. Fully simulated in Digital, not yet built.

### BX-4.3
A further refinement of BX-4.2 with cleaner Digital simulation files and machine code stored in Digital's native ROM format for easier loading and testing. Fully simulated in Digital, not yet built.

---

## What Is in the Repo

```
simple-4bit-cpu/
├── src/
│   ├── V1/          BX-4.1: Arduino ROM simulator code and Digital simulation file
│   ├── V2/          BX-4.2: Digital files, assembler (by Bolan Xu), assembly and machine code
│   ├── V3/          BX-4.3: Digital files and machine code in Digital ROM format
│   └── logbook/     Hand-written notes and log from the build process
├── images/          Photos of the BX-4.1 physical build
└── README.md
```

---

## Simulating in Digital

1. Download [Digital by H. Neemann](https://github.com/hneemann/Digital)
2. Open any `.dig` file from `src/V1`, `src/V2`, or `src/V3`
3. Run the simulation and step through clock cycles to trace execution

---

## Lessons Learned the Hard Way

**Propagation delay** is a real problem at this scale. Every gate takes a tiny amount of time to switch, and when you have long chains of logic those delays add up. Racing signals happen when two signal paths that should arrive at the same time are slightly out of sync, causing the wrong values to get latched. The fix was adding deliberate timing delays at the right spots in the circuit.

**Power distribution** was the biggest physical headache on the BX-4.1 build. You cannot run a single wire from chip to chip down the board like a chain. Every chip needs its own separate wire going directly back to the power supply, all in parallel. I found this out when I was measuring 5V right at the supply and only 4.3V at the chips on the far end of the board. That 0.7V drop was enough to make things act unreliably. Wire power in parallel, not in series.

---

## Tools and Resources

| Tool | Purpose |
|---|---|
| [Digital by H. Neemann](https://github.com/hneemann/Digital) | Logic simulation |
| Arduino | ROM and program counter simulation |
| 74-series TTL ICs | Physical logic implementation (BX-4.1) |
| *Code* by Charles Petzold | The book that started all of this |

---

## Credits

**Bohan Xu** — CPU design, schematic, physical build, simulation  
**[Bolan Xu](https://github.com/bolanxu)** — Assembler for BX-4.2 ([@TheBitDude](https://www.youtube.com/@TheBitDude) on YouTube)

---

## What Comes Next

The BX-4 was the starting point. After it came an 8-bit CPU (separate repository), and currently in active development is the **[APEX-16](https://github.com/BohanXu-74/APEX-16)**, a custom 16-bit pipelined CPU with a floating point unit inspired by Intel i386 through Pentium era architecture. It has a 6-stage pipeline (Fetch, Decode, Register Read, ALU/FPU, Memory, Register Write), around 8 general purpose registers, an ALU with AND, OR, NAND, NOT, XOR, shifts, and comparisons, and an FPU handling floating point addition and subtraction. Every problem that came up building the BX-4, from racing signals to power wiring, fed directly into how that design is being approached.

---

## License

MIT. See [LICENSE](LICENSE) for details.
