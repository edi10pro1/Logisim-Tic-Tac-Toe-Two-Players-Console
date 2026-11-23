# Hardware Tic-Tac-Toe Engine — Digital Logic & Datapath Design

A hardware-level implementation of the classic **Tic-Tac-Toe** game designed entirely from scratch using digital logic gates, sequential state machines, and dual-port memory in **Logisim-evolution (v4.1.0)**.

Built as a Computer Architecture / Digital Design project, this system explores how game state, user input conditioning, memory arbitration, and parallel pattern matching can be executed purely in hardware without a CPU or software instruction execution.

## Motivation & Engineering Challenges

When transitioning from software algorithms to pure hardware design, simple tasks like checking a 3x3 grid or registering a button press require careful timing and state management. Key hardware challenges solved in this project include:

1. **Eliminating Combinational Glitches During Memory Sweeps:** Reading cells sequentially from RAM to evaluate a win condition can cause transient false positives. I solved this by implementing a **two-stage shift-register pipeline (`SRG10` $\rightarrow$ `SRG9`)** gated by controlled tri-state buffers, latching a stable snapshot of the board only after a complete frame sweep.
2. **Read/Write Domain Decoupling:** Using a **16x2 Dual-Port RAM** allowed separating the user interaction logic (writing moves) from the continuous display/evaluation sweep logic without bus contention.
3. **Deterministic Startup & Reset:** Rather than relying on manual memory clearing, I designed a dedicated hardware **Initialization FSM (`boot_restart_routine`)** that automatically sweeps and formats the memory space upon system reset.

## System Architecture & Sub-Circuits

The top-level circuit (`main`) is structured into modular, reusable hardware blocks:

* **Memory & Cell Encoding (`DualRAM`):**
  * 16x2 Dual-Port RAM with asynchronous read enabled (`asyncread = true`).
  * Cells are encoded using a 2-bit representation: `00` (Empty / `-`), `01` (Player `X`), `10` (Player `O`), and `11` (Frame boundary/control marker).
* **Turn Control & Overwrite Protection (`write_and_ow_protection_unit`):**
  * Inspects the target cell before asserting `real_write`, blocking illegal moves on already-occupied positions.
  * Uses a **D Flip-Flop** toggle to alternate turns automatically between `01` (`X`) and `10` (`O`) after each valid move, while triggering an `auto_scroll` pulse to advance the cursor.
* **Boot & Reset Controller (`boot_restart_routine`):**
  * A **3-bit Counter + 3-to-8 Decoder** finite state machine triggered by `sys_rst`.
  * Generates timed control strobes (`set_addr`, `set_data`, `rst_routine_enable`) to clear RAM and reset display pointers cleanly.
* **Sample-and-Hold Serialization Pipeline (`SRG10` & `SRG9`):**
  * `SRG10` shifts in serial data swept from RAM. Once a full board frame is captured, tri-state buffers transfer the 9 cell values in parallel into `SRG9`, providing a glitch-free bus to the win-detection engine.
* **Parallel Win-Detection Engine (`check_line`):**
  * **8 parallel combinational blocks** evaluate all 3 rows, 3 columns, and 2 diagonals simultaneously.
  * Each block computes a 2-bit bitwise `AND` across three cells, aggregated via an **8-input 2-bit `OR` gate** (`win_sym`). A final `XOR` gate asserts the global `win` flag strictly when a valid player symbol (`01` or `10`) completes a line.
* **Input Conditioning & TTY Display (`one_shot_unit` & `ASCII_decoder`):**
  * `one_shot_unit`: Edge-detector circuits built with D Flip-Flops that convert asynchronous button presses (`scroll`, `write`, `sys_rst`) into clean, single-cycle pulses.
  * `ASCII_decoder` & Victory Override: Maps 2-bit cell values into 7-bit ASCII codes (`0x2D`, `0x58`, `0x4F`) for a 3x3 TTY display. When `win` goes high, a 2-to-1 MUX overrides the RAM sweep and floods the screen with the winning player's symbol.

## Running the Simulation

1. Open `PROJECT.circ` in **[Logisim-evolution v4.1.0](https://github.com/logisim-evolution/logisim-evolution)** (or newer).
2. Enable auto-ticking (`Simulate -> Auto-Tick Enabled`, recommended frequency: `64 Hz`).
3. Pulse `sys_rst` once to trigger the hardware initialization routine, then use the `scroll` and `write` push-buttons to play.

## Key Takeaways

Through this project, I gained hands-on experience in:
* Designing modular datapaths and control units from basic logic gates and flip-flops.
* Managing clock-domain synchronization, edge detection, and tri-state bus control.
* Debugging timing hazards and race conditions using waveform/state inspection in simulation.