# TOY-16 CPU in Logisim Evolution

A 16-bit TOY computer built and simulated in Logisim Evolution. The project extends the TOY-8 design with a wider datapath, a larger memory, a 16-register register file, and a four-phase fetch/execute controller.

> **Project status:** The CPU backend and front panel are assembled. The screenshots below document the current circuit design and an example front-panel state.

## Architecture at a glance

| Component | Design |
|---|---|
| Word size | 16 bits |
| Main memory | 256 words × 16 bits |
| Address size | 8 bits |
| General-purpose registers | 16 registers, 16 bits each |
| Register-file access | Two independent reads and one shared write |
| Instruction word | 16 bits |
| Timing | Fetch, Fetch-Write, Execute, Execute-Write |

## Instruction formats

Every instruction is 16 bits and begins with a 4-bit opcode and a 4-bit destination/register field `d`.

- **Format RR:** `opcode (4) | d (4) | s (4) | t (4)` — register-to-register operations use `Rs` and `Rt`.
- **Format A:** `opcode (4) | d (4) | addr (8)` — memory, immediate, and control-flow operations use an 8-bit address or value.

The address field is zero-extended when connected to 16-bit data paths.

## Instruction set

| Opcode | Instruction | Description |
|---|---|---|
| `0` | HALT | Stop execution. |
| `1` | ADD | Add register operands. |
| `2` | SUBTRACT | Subtract register operands. |
| `3` | AND | Bitwise AND. |
| `4` | XOR | Bitwise XOR. |
| `5` | SHIFT LEFT | Shift left. |
| `6` | SHIFT RIGHT | Shift right. |
| `7` | LOAD ADDRESS | Load the address/immediate field into a register. |
| `8` | LOAD | Load from a direct memory address. |
| `9` | STORE | Store a register value to a direct memory address. |
| `A` | LOAD INDIRECT | Load from the memory address held in a register. |
| `B` | STORE INDIRECT | Store to the memory address held in a register. |
| `C` | BRANCH ZERO | Branch when the selected register is zero. |
| `D` | BRANCH POSITIVE | Branch when the selected register is positive. |
| `E` | JUMP REGISTER | Jump to the address held in a register. |
| `F` | JUMP / LINK | Jump and save the return address in a register. |

## How the CPU works

### Four-phase controller

A clock drives a 2-bit counter and a 2-to-4 decoder to produce one-hot timing phases:

1. **Fetch:** The program counter selects a memory word and the memory output settles.
2. **Fetch-Write:** The instruction register captures the word and the program counter advances.
3. **Execute:** The opcode and register values drive the control unit and datapath.
4. **Execute-Write:** The result is committed to a register, memory, or the program counter.

### Register file

The register file uses two 16 × 16 RAM components as mirrored storage. They share write data and write control, while each provides a separate read port for `Rs` and `Rt`. A safe-write circuit blocks writes to `R0`, which is reserved to remain zero. A read-address mux lets store and branch operations read `Rd` when they need it.

### Datapath and write-back

The ALU handles arithmetic, logic, shifts, and pass-through operations. A write-back mux selects the result source. The selected value passes through a 16-bit buffer register before it is written to the register file, preserving the result while the register-file address changes for the write.

The program counter is 8 bits wide. Its input path selects between the incremented PC and a branch or jump target.

## Front panel

The `Front_Panel_TOY16` circuit provides manual controls for `LOOK`, `LOAD`, `RUN`, `RESET`, and `STEP`, along with global clock, panel address, and panel data inputs. Its displays expose internal values such as `RS_DATA_OUT`, `RT_DATA_OUT`, `RAM_ADDR_OUT`, `ALU_OUT`, `RAM_MEM_OUT`, `PC_OUT`, and `IR_OUT` for observation while simulating.

## Circuit screenshots

### CPU backend overview

![TOY-16 CPU backend](images/cpu-backend-overview.png)

### Front panel

![TOY-16 front panel](images/front-panel-overview.png)

### Register file with two read ports

![Mirrored dual-read register file](images/register-file-dual-read.png)

### Write-back path and buffer register

![Write-back mux and 16-bit buffer register](images/writeback-buffer.png)

### Program-counter load mux

![Program-counter load mux](images/pc-load-mux.png)

### Building a 16-bit register from two 8-bit registers

![16-bit register assembled from two 8-bit registers](images/register-16-bit-from-two-8-bit-registers.png)

### Front-panel simulation state

![Front panel showing a simulation state](images/front-panel-state.png)

## Opening and using the project

1. Open the `.circ` project in Logisim Evolution.
2. Open `Front_Panel_TOY16` to use the manual controls and observe the CPU outputs.
3. Load instruction words into the memory component using the project's memory editor.
4. Use reset and the front-panel clock/step controls to inspect fetch and execute behavior.

The project circuit file should be committed alongside this README. Keep the `images/` folder at the same repository level as `README.md` so the screenshots render on GitHub.

## Design notes

- `R0` is protected from writes and is intended to remain `0000`.
- The 8-bit address field must be zero-extended when it feeds a 16-bit data input.
- The register-file write-back buffer is included to avoid losing the ALU result during a write-address change.
- The four timing phases separate memory reads, instruction capture, execution, and state updates.
