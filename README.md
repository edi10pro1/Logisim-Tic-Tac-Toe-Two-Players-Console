# Hardware Tic-Tac-Toe Engine (Digital Logic Design)

A complete, hardware-level implementation of the classic **Tic-Tac-Toe** game designed from scratch using pure digital logic in **Logisim-evolution (v4.1.0)**[cite: 6].

![Logisim-evolution](https://img.shields.io/badge/Simulated_in-Logisim--evolution_v4.1.0-blue)
![Architecture](https://img.shields.io/badge/Design-FSM_%26_Combinational_Logic-green)
![Topic](https://img.shields.io/badge/Course-Computer_Architecture_%26_Digital_Design-orange)

---

## 🎯 Project Overview

Unlike software implementations that rely on a CPU or microcontroller executing sequential code, this project implements the entire game state, turn management, memory synchronization, and win-detection algorithm directly using **digital logic gates, flip-flops, counters, and memory blocks**.

The design strictly separates **sequential control/memory paths** from **parallel combinational evaluation**, illustrating core Computer Architecture concepts such as data-path control, bus multiplexing, edge-triggered synchronization, and pipelined data stabilization.

---

## 🏗️ System Architecture & Sub-Circuits

The top-level circuit (`main`) is modularized into specialized functional units[cite: 6]:

### 1. Dual-Port Memory & Cell Encoding (`DualRAM`)
* Uses a **16x2 Dual-Port RAM** with asynchronous read capabilities (`asyncread = true`)[cite: 6].
* Decouples the **player write/interaction domain** from the **display and win-evaluation sweep domain**.
* **2-Bit Cell State Encoding:**
  * `00` (`0x0`): Empty cell (`-`)
  * `01` (`0x1`): Player 1 (`X`)
  * `10` (`0x2`): Player 2 (`O`)
  * `11` (`0x3`): Control / End-of-frame delimiter

### 2. Turn Management & Overwrite Protection (`write_and_ow_protection_unit`)
* Prevents illegal moves by verifying that the currently addressed cell is empty (`00`) before asserting `real_write`[cite: 6].
* Uses a **D Flip-Flop** toggle to automatically alternate turns between Player `01` (`X`) and Player `10` (`O`) upon every valid move[cite: 6].
* Generates an `auto_scroll` pulse immediately after a valid placement to advance the cursor to the next cell[cite: 6].

### 3. Hardware Initialization & Reset FSM (`boot_restart_routine`)
* A dedicated Finite State Machine built with a **3-bit Counter and a 3-to-8 Decoder**, triggered by `sys_rst`[cite: 6].
* Automatically sweeps through memory addresses, clears the board state, and resets display pointers (`set_addr`, `set_data`, `rst_routine_enable`) before handing control back to the players[cite: 6].

### 4. Serialization & Snapshot Pipeline (`SRG10` & `SRG9`)
* A **10-stage Shift Register (`SRG10`)** sweeps the serial data stream coming out of the Dual-Port RAM[cite: 6]