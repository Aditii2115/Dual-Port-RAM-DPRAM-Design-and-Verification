# Dual-Port-RAM-DPRAM-Design-and-Verification

## Overview

This project presents the design and verification of a **True Dual-Port Random Access Memory (DPRAM)** that supports simultaneous read and write operations through two independent ports — **Port A** and **Port B**. The design enables concurrent memory access, improving system performance and data throughput in applications requiring parallel processing.

Each port operates with its own address, data input, data output, and write enable control signals, allowing fully independent operation. A key feature of the design is its **deterministic priority-based conflict resolution mechanism**: when both ports attempt to write to the same memory address simultaneously, Port A is given priority, ensuring predictable and consistent behavior.

The design was verified using a structured SystemVerilog/Verilog testbench covering independent operations, simultaneous access, and same-address conflict scenarios, with simulation performed in **ModelSim**.

## Features

- True dual-port memory supporting simultaneous read/write via two independent ports
- Deterministic **Port A priority** conflict resolution on same-address write collisions
- Fully synchronous RTL design (single clock, rising-edge triggered)
- Independent write-enable control for each port
- Automated testbench with **error counters** for pass/fail verification
- Verified against reset, independent operation, simultaneous access, conflict, and boundary test scenarios

## Design Parameters

| Parameter | Value |
|---|---|
| Memory Depth | 8–20 locations (addresses 0 to 7 / 0 to 19 depending on config) |
| Data Width | 8 bits per location |
| Address Width | 3 bits (8 locations) |
| Clock | Single synchronous clock, shared by both ports |
| Write Enables | Independent (`we_a`, `we_b`) for Port A and Port B |

## Architecture

The memory is implemented as a register array (`mem[0:N-1]`), with each port having its own address, data, and control lines feeding into shared memory cells, arbitrated by conflict-resolution logic.

```
CPU/Device "A" -> [Address A | Data A | R/W A] -> Port A Logic -+
                                                                  |-> Dual-Port RAM Memory Cells
CPU/Device "B" -> [Address B | Data B | R/W B] -> Port B Logic -+
```

**Operation logic:**
- If `we_a = 1` -> data is written to `mem[add_a]`
- If `we_b = 1` and `add_b != add_a` (or Port A isn't writing) -> data is written to `mem[add_b]`
- If `add_a == add_b` and both write enables are active -> **only Port A writes**; Port B's write is ignored
- Both ports can read independently (`read_a = mem[add_a]`, `read_b = mem[add_b]`) every clock cycle

## Verification Strategy

The testbench validates the design across the following scenarios:

1. **Reset Verification** — confirms memory and registers initialize correctly
2. **Independent Operations** — write/read via Port A only, and via Port B only
3. **Simultaneous Operations** — concurrent reads, concurrent writes to different addresses, one port reading while the other writes
4. **Conflict Testing** — both ports writing to the same address, confirming Port A priority
5. **Boundary Testing** — lowest and highest memory addresses

**Error Detection:** Two counters, `error1` (Port A mismatches) and `error2` (Port B mismatches), track discrepancies between expected and actual outputs during simulation. The design is considered correct only if `error1 + error2 == 0`.

## Simulation Results

Simulated in **ModelSim (Altera Starter Edition 10.1d)**, covering:

- Reset operation
- Write A / Read A
- Write B / Read B
- Simultaneous write to the same address (Port A and Port B) followed by read verification confirming Port A's data is retained

All test scenarios passed with zero mismatches, confirming correct functional behavior of the True Dual-Port RAM, including proper conflict resolution.

## Repository Contents

- `dual.v` — RTL design of the Dual-Port RAM
- `tb_dual.v` — Self-checking testbench with automated error counters
- Simulation waveform screenshots (ModelSim)
- Full report (PDF) with detailed methodology, block diagram, flowchart, and results

## How to Run

1. Open the project in **ModelSim** (or any Verilog simulator).
2. Compile `dual.v` (RTL) and `tb_dual.v` (testbench).
3. Run the simulation — the testbench applies reset, performs independent and simultaneous write/read operations, and checks for conflicts.
4. Check the console output: `ALL TESTS PASSED - Memory working correctly.` indicates a successful verification run.

## Applications

- High-performance digital systems requiring parallel memory access
- FPGA-based communication and processing systems
- Educational demonstration of memory arbitration and RTL verification techniques



Key references include works by Cummings (FIFO/memory design techniques), Sutherland et al. (SystemVerilog for hardware design), Bergeron (verification methodologies), and Harris & Harris (digital design and computer architecture). See the full report for complete citations.
