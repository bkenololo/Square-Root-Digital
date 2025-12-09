# 16-bit Fixed-Point Square Root Accelerator on FPGA
A high-throughput hardware accelerator for calculating square roots using the Newton-Raphson method and Goldschmidt's Division, implemented on Cyclone IV FPGA.

## 🚀 Overview
This project implements a hardware-based square root calculator designed for high-speed digital signal processing applications. Unlike standard iterative approaches, this system leverages specific hardware optimizations to balance precision, latency, and resource utilization.

* **Input/Output:** 16-bit Unsigned Integer / Fixed-Point **U-Q8.8**.
* **Target Hardware:** Altera/Intel Cyclone IV FPGA.
* **Key Algorithms:** Newton-Raphson Approximation & Goldschmidt Division.

## ✨ Key Features
* **Fixed-Point Arithmetic:** Optimized for **U-Q8.8** format to minimize floating-point overhead.
* **High-Speed Division:** Utilizes **Goldschmidt’s Algorithm** to replace standard long division, taking advantage of the FPGA's integrated **18x18 DSP Multipliers**.
* **Efficient Normalization:** Implements a **Binary Tree MSB Detector** and **Barrel Shifter** for low-latency data normalization.
* **Robust Control:** Dedicated **FSMs** for UART communication and Main Control, featuring a **Check-Zero failsafe** to prevent division-by-zero errors.
* **Interface:** Custom **16-bit UART RX/TX** for real-time data acquisition.

## 🏗️ System Architecture
### 1. Datapath
The datapath is designed to support the iterative nature of the Newton-Raphson method ($x_{i+1} = 0.5 \times (x_i + N/x_i)$).
* **Multiplication:** Uses on-chip **DSP Blocks** for fast execution.
* **Initial Guess:** Retrieved from a **BRAM-based Look-Up Table (LUT)** to reduce convergence time.
* **Normalization:** A combinational logic block (MSB Detector + Barrel Shifter) scales the input to the optimal range for the algorithm.

### 2. Control Logic (FSM)
The system is governed by a central Finite State Machine with the following states:
1.  **IDLE:** Waiting for valid input.
2.  **CHECK ZERO:** Failsafe state to return 0 immediately if input is 0.
3.  **LUT LOOKUP:** Fetching the initial seed.
4.  **DIVIDE:** Executing Goldschmidt division iterations.
5.  **CHECK:** Verifying convergence criteria.
6.  **DONE:** Outputting valid result via UART.

## 📊 Algorithm Details
### Newton-Raphson Method
Used for finding the root. We implemented the fixed-point variant to allow for iterative convergence without floating-point units.

### Goldschmidt Division
Selected over restoring/non-restoring division because it converges quadratically and can be parallelized using multipliers, making it significantly faster on FPGAs with DSP slices.

## 🛠️ Tools & Technologies
* **Language:** VHDL
* **Simulation:** ModelSim / Questasim
* **Synthesis:** Intel Quartus Prime
* **Hardware:** DE10-Lite / Cyclone IV Board
