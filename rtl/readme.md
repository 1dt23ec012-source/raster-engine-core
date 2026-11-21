# Synthesizable RTL Architecture (SystemVerilog)

This directory contains the synthesizable SystemVerilog source files for the vendor-agnostic 2D graphics raster engine core (`raster_core`).

## Module Breakdown

| Module File | Functional Description | Architecture & Datapath Role |
| :--- | :--- | :--- |
| **`raster_core.sv`** | Top-Level Engine Subsystem | Top-level integration uniting command parser, primitive rasterizers, pixel coalescer, and output FIFOs. |
| **`cmd_parser.sv`** | Command Stream Decoder | FSM decoding incoming packet opcodes, validating bounding coordinates, and extracting color attributes. |
| **`cmd_exec.sv`** | Execution Sequencer | Master controller managing primitive draw dispatch, front/back framebuffer toggles, and completion handshakes. |
| **`rect_line.sv`** | Bresenham Line & Rect Unit | Dedicated hardware engine for integer Bresenham line drawing and high-speed axis-aligned rectangle fills. |
| **`tri_unit.sv`** | Triangle Rasterizer | Barycentric coordinate evaluator evaluating edge functions for solid filled triangle primitives. |
| **`pix_coalescer.sv`** | Memory Write Coalescer | Accumulates individual pixel writes into aligned 64-pixel bursts to maximize external memory bus bandwidth. |
| **`fifo.sv`** | Synchronous Parameterized FIFO | Parameterized circular synchronous FIFO (`fifo_sync`) sized to 128 entries to minimize flip-flop footprint. |
| **`demo_gen.sv`** | Autonomous Pattern Generator | Standalone procedural pattern generator producing geometric sweeps for hardware bring-up and display diagnostics. |

## Sequential Logic Optimization
* Both the transmit queue and pixel data FIFO depths are parameterized to **128 entries** (address width 7).
* Sizing the queue to 128 entries comfortably accommodates the 64-pixel maximum burst of the coalescer while slashing sequential flip-flop count from **20,291 down to 11,067** (a **45% reduction** in sequential cells).
