<div align="center">

# 2D Graphics Raster Engine Core

### **Synthesizable 2D Graphics Accelerator | SystemVerilog | AMD-Xilinx Artix-7**
**Verification:** 15-Seed Verilator Random Regression against Python Pixel-Exact Golden Model  
**Toolchain:** SystemVerilog (Slang frontend), Verilator 5.044, AMD-Xilinx Vivado

[![FPGA](https://img.shields.io/badge/FPGA-Artix--7%20(XC7A35T)-purple?style=for-the-badge&logo=amd&logoColor=white)](https://github.com/1dt23ec012-source/raster-engine-core)
[![Verilator](https://img.shields.io/badge/Verification-Verilator%205.044-informational?style=for-the-badge)](https://github.com/1dt23ec012-source/raster-engine-core)
[![Language](https://img.shields.io/badge/Language-SystemVerilog-blue?style=for-the-badge&logo=cplusplus&logoColor=white)](https://github.com/1dt23ec012-source/raster-engine-core)
[![Status](https://img.shields.io/badge/Regression-100%25%20Passing-brightgreen?style=for-the-badge)](https://github.com/1dt23ec012-source/raster-engine-core)

</div>

---

## 📖 Executive Summary

The **Raster Engine Core** (`raster_core`) is a fully synthesizable hardware graphics rendering pipeline implemented in SystemVerilog. It decodes high-level geometric draw commands, executes hardware-accelerated Bresenham line/rectangle generation and barycentric triangle rasterization, coalesces pixel writes into memory bursts, and streams frame data through high-throughput synchronous FIFOs.

The core was architected to be modular and vendor-agnostic, operating on a single synchronous clock domain. To verify functional correctness across all edge cases, the SystemVerilog implementation was verified against a **cycle-accurate Python reference model** using an automated randomized regression suite in **Verilator**.

---

## 🏛️ Pipeline Architecture

```text
                                raster_core Subsystem
┌──────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                                                                                  │
│   Command Stream             ┌─────────────────┐       Primitive Draw Commands                   │
│  ───────────────────────────►│   cmd_parser    ├───────────────────────┐                         │
│                              └────────┬────────┘                       │                         │
│                                       │                                ▼                         │
│                              ┌────────▼────────┐            ┌────────────────────┐               │
│                              │    cmd_exec     │            │ rect_line / tri_unit│              │
│                              └────────┬────────┘            └──────────┬─────────┘               │
│                                       │                                │ Rendered Pixels         │
│                              ┌────────▼────────┐                       ▼                         │
│                              │    demo_gen     │            ┌────────────────────┐               │
│                              └─────────────────┘            │   pix_coalescer    │               │
│                                                             └──────────┬─────────┘               │
│                                                                        │ 64-Pixel Bursts         │
│   Transmit Queue (128x8)                            Pixel FIFO (128x16)▼                         │
│  ┌───────────────────────┐                         ┌──────────────────────────────────┐          │
│  │       fifo_sync       │                         │            fifo_sync             │          │
│  └───────────────────────┘                         └──────────────────────────────────┘          │
│                                                                                                  │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
