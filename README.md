# Simple 4-Bit CPU (LIMP)

[![Demo Video](https://img.shields.io/badge/YouTube-Demo%20Video-red?logo=youtube)](https://www.youtube.com/watch?v=geT2ecY6J6I)
[![Hackaday](https://img.shields.io/badge/Hackaday.io-Project%20Page-brightgreen)](https://hackaday.io/project/204999-student-made-ttl-cpu)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

![CPU Photo](images/cpu.png)

A real, physical 4-bit CPU built from 74-series TTL logic chips, designed and assembled in 5th grade after reading *Code* by Charles Petzold. This is not a simulator or a paper design. It actually runs. Every instruction is traceable cycle by cycle through discrete logic gates wired on a real board.

This repo contains three generations of the design (V1, V2, V3), the Digital simulation files, the Arduino-based ROM simulator code, assembly programs, a hand-written log book, and a custom assembler written by [Bolan Xu](https://github.com/bolanxu).

---

## Background

After reading *Code* by Charles Petzold, I wanted to actually understand how a processor works from the ground up, not just read about it. So I built one. The result is what I call LIMP, a simple 4-bit CPU implemented entirely in 74-series TTL logic.

The whole thing is closer to a microcontroller than a true microprocessor. There is no external RAM access. The program lives in ROM simulated by an Arduino, and the CPU executes it instruction by instruction. It is slow, it has timing quirks, and getting it to actually run reliably was a real fight, but it works.

This project directly led to my 8-bit CPU (in a separate repo), and is part of a longer journey that now includes the [APEX-16](#what-comes-next), a 16-bit pipelined CPU with an FPU that I am actively building.

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

The 12-bit program counter gives the CPU 4096 addressable locations in program memory, which is a lot more than this architecture needs but gives room to work with.

The four registers (R1 through R4) are all general purpose. There is no dedicated accumulator or stack pointer at this level. You move data between registers, load immediate values, do arithmetic, compare, and jump. That is the whole model.

---

## Instruction Set

The CPU supports 15 instructions total. The core ones are:

| Instruction | Description |
|---|---|
| `MOV #num, Rn` | Load an immediate 4-bit value directly into a register |
| `MOV Rn, Rm` | Copy the contents of one register into another |
| `ADD Rn, Rm` | Add two registers, result stored back |
| `CMP Rn, Rm` | Compare two registers and set flags |
| `JMP addr` | Unconditional jump to a 12-bit address |
| `COMPLEMENT Rn` | Bitwise invert a register (NOT operation) |

The ALU itself only does addition and complement natively. Subtraction can be done through complement and add, which is the standard two's complement method that real processors use too.

---

## How the ROM Simulator Works

Instead of a physical ROM chip, the program memory is simulated by an Arduino. On each clock pulse, the Arduino outputs the current instruction to the CPU's data bus. It also simulates the program counter externally, tracking which instruction comes next.

When a `JMP` instruction is executed, the CPU latches the 12-bit jump target address directly onto 12 of the Arduino's I/O pins. The Arduino reads that address and jumps its internal program counter to match, so the next clock pulse outputs the instruction at the new location.

You can also write programs directly into the Arduino's memory, making it easy to change what the CPU runs without rewiring anything.

---

## Versions

### V1
The first working design. Everything is functional but rough. Timing issues and power problems are present. Logic was designed and verified in Digital by H. Neemann before being built physically.

### V2
A significant redesign informed by techniques from the 8051 architecture. Instructions execute in fewer clock cycles compared to V1. This version also includes a custom assembler written by [Bolan Xu](https://github.com/bolanxu) ([@TheBitDude](https://www.youtube.com/@TheBitDude)), along with hand-written assembly and machine code examples.

### V3
Further refinement of V2. The Digital simulation files are more complete, and machine code is stored in Digital's native ROM format for easier loading and testing.

---

## What Is in the Repo

```
simple-4bit-cpu/
├── src/
│   ├── V1/          Arduino ROM simulator code and Digital simulation file
│   ├── V2/          V2 Digital files, assembler (by Bolan Xu), assembly and machine code
│   ├── V3/          V3 Digital files and machine code in Digital ROM format
│   └── logbook/     Hand-written notes and log from the build process
├── images/          Photos of the physical CPU
└── README.md
```

---

## Simulating in Digital

1. Download [Digital by H. Neemann](https://github.com/hneemann/Digital)
2. Open any of the `.dig` files from the `src/V1`, `src/V2`, or `src/V3` folders
3. Run the simulation and step through clock cycles to trace execution

---

## Lessons Learned the Hard Way

**Propagation delay** is a real problem at this scale. Every gate takes a small amount of time to switch, and when you chain a lot of them together those delays add up. Racing signals happen when two different signal paths that are supposed to arrive at the same time are slightly out of sync, which causes the wrong values to get latched. The fix was adding deliberate timing delays at the right spots in the circuit.

**Power distribution** was the biggest physical headache. You can not just run a single wire from chip to chip to chip down the board. Every chip needs its own separate wire going directly back to the power supply in parallel. I found this out the hard way when I was measuring 5V right at the supply and only 4.3V at the chips on the far end of the board. That 0.7V drop was enough to make things act weird. The fix is simple once you know it: wire all the chips to power in parallel, not in a chain.

---

## Tools and Resources

| Tool | Purpose |
|---|---|
| [Digital by H. Neemann](https://github.com/hneemann/Digital) | Logic simulation |
| Arduino | ROM and program counter simulation |
| 74-series TTL ICs | Physical logic implementation |
| *Code* by Charles Petzold | The book that started all of this |

---

## Credits

**Bohan Xu** — CPU design, schematic, physical build, simulation  
**[Bolan Xu](https://github.com/bolanxu)** — Assembler for V2 ([@TheBitDude](https://www.youtube.com/@TheBitDude) on YouTube)

---

## What Comes Next

This CPU was the first step. After finishing it, I built an 8-bit CPU (separate repository). I am currently working on the **[APEX-16](https://github.com/BohanXu-74/APEX-16)**, a custom 16-bit pipelined CPU with a floating point unit inspired by the Intel i386 through Pentium era of architecture. It has a 6-stage pipeline (Fetch, Decode, Register Read, ALU/FPU, Memory, Register Write), around 8 general purpose registers, and an ALU that handles AND, OR, NAND, NOT, XOR, shifts, and comparisons. The FPU handles floating point addition and subtraction. It is still in active development, but everything I ran into on this 4-bit CPU including the timing problems and the wiring mistakes fed directly into how I am approaching that design.

---

## License

MIT. See [LICENSE](LICENSE) for details.
