# FPGA Fixed-Point Square Root Accelerator (Q8.8)

##  TL;DR

* **Function:** Hardware accelerator for Square Root () calculation using **Newton-Raphson** & **Goldschmidt Division**.
* **Precision:** 16-bit Fixed-Point (**U-Q8.8** format).
* **Performance:** **SQNR 79.63 dB** (Equivalent to ~13-bit precision).
* **Verification:** Python-based Hardware-in-the-Loop (HIL) via **UART**.
* **Hardware:** Target for Altera Cyclone IV (DE10-Lite).

---

## 📂 Repository Structure

```text
.
├── src/           # Synthesizable VHDL Source Code (RTL)
│   ├── System_Integration_Top.vhd   # Top-Level Entity
│   ├── Sqrt_Logic.vhd               # Core Arithmetic Logic
│   └── UART_*.vhd                   # UART TX/RX Modules
│
├── tb/            # VHDL Testbenches
│   └── tb_sqrt.vhd                  # Simulation Testbench
│
├── script/        # Python Verification Suite
│   ├── full_test_suite.py           # Automated HIL Testing
│   └── generate_report.py           # Statistical Analysis
│
├── data/          # Measurement Results
│   ├── scientific_results.csv       # Precision Analysis
│   └── latency_results.csv          # Timing Analysis
│
└── docs/          # Documentation
    ├── Block_Diagram.png
    └── Waveform_Simulation.png

```

---

## ⚙️ Technical Specifications

| Parameter | Value / Description |
| --- | --- |
| **Numeric Format** | Unsigned Fixed-Point Q8.8 (8-bit Integer, 8-bit Fractional) |
| **Input Range** |  (16-bit integer mapped to Q8.8) |
| **Core Algorithm** | Iterative Newton-Raphson () |
| **Division Method** | Goldschmidt Algorithm (Multiplication-based convergence) |
| **Multiplier** | Utilizes FPGA Embedded **DSP Blocks (18x18)** |
| **Communication** | UART (Configurable Baud: 9600 / 115200) |
| **Resource Usage** | Optimized for Logic Elements (LEs) & Embedded Multipliers |

---

## 📊 Measured Performance (Hardware Validation)

*Based on 65,535 test vectors verified against Python `math.sqrt` reference.*

### Precision & Accuracy

* **SQNR:** **79.63 dB** (Signal-to-Quantization-Noise Ratio).
* **ENOB:** **12.94 Bits** (Effective Number of Bits).
* **Max Error:** 8.0 LSB (Due to integer truncation logic).
* **Pass Rate:** 100% (within hardware truncation tolerance of < 10 LSB).

### Timing & Latency

* **Average Latency:** 7.67 ms (Round-trip @ 9600 baud).
* **Processing Time:** Negligible (< 10 s internal FPGA processing).
* **Throughput:** ~130 operations/sec (Limited by UART Baud Rate).

---

## 🏗️ Architecture Details

### 1. Datapath Pipeline

The arithmetic core avoids costly floating-point units by strictly using integer operations:

* **Normalization:** Binary Tree MSB Detector + Barrel Shifter normalizes input to range .
* **Initial Guess:** 8-entry Look-Up Table (LUT) provides accurate seed to reduce iteration count.
* **Iteration Core:**
* **Goldschmidt Divider:** Replaces long-division with iterative multiplication, leveraging Cyclone IV DSP blocks.
* **Newton-Raphson Step:** Performs the update .



### 2. Control Unit (FSM)

A centralized Finite State Machine orchestrates the data flow:

* **IDLE:** Awaits `rx_done` signal from UART.
* **CHECK_ZERO:** Single-cycle bypass for Input=0.
* **NORMALIZE & LUT:** Pre-processing for convergence.
* **CALC_LOOP:** Executes fixed iteration cycles.
* **DENORMALIZE:** Rescales result back to original magnitude.
* **TX_SEND:** Transmits result via UART.

---

## 🚀 How to Run

### Simulation (ModelSim)

1. Open ModelSim/Questa.
2. Compile all files in `src/` and `tb/`.
3. Run simulation on `tb_sqrt`.

### Hardware Verification

1. Open Quartus Prime and compile the project.
2. Program the `.sof` file to the FPGA.
3. Connect FPGA UART to PC.
4. Run the verification script:
```bash
pip install pyserial pandas
python script/full_test_suite.py

```


5. Check `data/FPGA_Professional_Report.csv` for results.
