# 8-Bit Microprocessor in CMOS

Yi-Hsiang Wei and Zijian Shang  
Department of Electrical Engineering, Columbia University  
VLSI Design Project

## Abstract

This report presents the layout, physical verification, and waveform-level functional verification of an 8-bit custom CMOS microprocessor. The processor integrates a PLA-based instruction decoder, control-signal latch, 8x8 SRAM, arithmetic datapath, shifter, accumulator latch, mux, and bidirectional external bus interface.

The instruction set supports eight operations: `NOP`, `LOAD`, `STORE`, `GET`, `PUT`, `ADD`, `SUB`, and `SHIFT`. In this version, schematic figures are intentionally omitted and will be inserted later. The included figures are limited to layout views, DRC/LVS verification captures, and transient waveform results.

## I. Introduction

The goal of this project is to design and verify a compact 8-bit microprocessor at the custom-layout level. The processor operates on 8-bit data words and uses a 3-bit opcode plus memory address bits to select memory, arithmetic, shift, and external bus operations.

The major top-level signals are summarized below.

| Signal | Direction | Description |
| --- | --- | --- |
| `PHI1`, `PHI2` | Input | Two-phase clock signals |
| `INSTR<0:5>` | Input | Opcode and memory address fields |
| `EXT_BUS<0:7>` | Bidirectional | External 8-bit data bus |
| `SHIFT_BYPASS` | Control | Selects shift-bypass mode |
| `C` | Output | Carry flag |
| `OV` | Output | Overflow flag |

<div align="center">
<img src="figures/report/fig01-top-level-layout.png" alt="Fig. 1. Top-level microprocessor layout." width="760"><br>
<em>Fig. 1. Top-level microprocessor layout.</em>
</div>

## II. Architecture

### A. Top-Level Datapath

The datapath connects the SRAM output, accumulator latch, arithmetic unit, shifter, multiplexer, and external bus driver. The control path decodes `INSTR<0:2>` into operation-specific control signals and latches timing-sensitive controls for datapath evaluation.

```text
SRAM -> input latch -> adder/subtractor -> mux -> accumulator latch -> bus driver
                         ^                  ^
                         |                  |
                      memory data        shifter output
```

The external bus can load memory during `LOAD` or receive stored data during `STORE`. Arithmetic operations use the accumulator and SRAM data as operands.

### B. Instruction Set and Control Table

The opcode and decoded control behavior are summarized in Table I.

| Instruction | Opcode | Function | `SUB` | `MUX2` | `MUX1` | `MUX0` | `MEM_WRITE` | `MEM_READ` | `DRV_EN` | `SHIFT_BYPASS` | `LOAD_BUS` | `STORE_BUS` |
| --- | --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| `NOP` | `000` | No operation | - | 0 | 0 | 1 | 0 | - | 0 | 0 | 0 | 0 |
| `LOAD` | `001` | `Mem[i] <- External Bus` | - | - | - | - | 1 | - | 0 | - | 1 | 0 |
| `STORE` | `010` | `External Bus <- Mem[i]` | - | 1 | 0 | 0 | 0 | 1 | 0 | - | 0 | 1 |
| `GET` | `011` | `Acc <- Mem[i]` | - | 1 | 0 | 0 | 0 | 1 | 0 | - | 0 | 0 |
| `PUT` | `100` | `Mem[i] <- Acc` | - | - | - | - | 1 | - | 1 | - | 0 | 0 |
| `ADD` | `101` | `Acc <- Acc + Mem[i]` | 0 | 0 | 1 | 0 | 0 | 1 | 0 | 0 | 0 | 0 |
| `SUB` | `110` | `Acc <- Acc - Mem[i]` | 1 | 0 | 1 | 0 | 0 | 1 | 0 | 0 | 0 | 0 |
| `SHIFT` | `111` | Shift accumulator left by `i` | - | 0 | 0 | 1 | 0 | - | 0 | 1 | 0 | 0 |

## III. Layout Implementation

### A. PLA Layout

The instruction decoder is implemented as a PLA driven by the opcode bits. The layout follows a regular row-column structure for the product terms and decoded control outputs.

<div align="center">
<img src="figures/report/fig04-pla-layout.png" alt="Fig. 2. PLA layout." width="700"><br>
<em>Fig. 2. PLA layout.</em>
</div>

### B. Control Latch Layout

The control latch stores the selected PLA outputs so that datapath control remains stable during evaluation. The block includes latch stages for subtraction, mux selection, and shift control.

<div align="center">
<img src="figures/report/fig05-control-latch-layout.png" alt="Fig. 3. Control-signal latch layout." width="760"><br>
<em>Fig. 3. Control-signal latch layout.</em>
</div>

<div align="center">
<img src="figures/report/fig06-control-latch-cell-layouts.png" alt="Fig. 4. Inverter and latch cell layouts used in the control latch." width="600"><br>
<em>Fig. 4. Inverter and latch cell layouts used in the control latch.</em>
</div>

### C. SRAM Layout

The SRAM stores eight 8-bit words. It includes address decoding, bit-line precharge, write circuitry, read circuitry, and the 8x8 memory array.

<div align="center">
<img src="figures/report/fig07-sram-layout.png" alt="Fig. 5. SRAM block layout." width="700"><br>
<em>Fig. 5. SRAM block layout.</em>
</div>

<div align="center">
<img src="figures/report/fig08-sram-decoder-layout.png" alt="Fig. 6. SRAM decoder layout." width="380"><br>
<em>Fig. 6. SRAM decoder layout.</em>
</div>

<div align="center">
<img src="figures/report/fig09-sram-precharge-layout.png" alt="Fig. 7. SRAM precharge circuit layout." width="600"><br>
<em>Fig. 7. SRAM precharge circuit layout.</em>
</div>

<div align="center">
<img src="figures/report/fig10-sram-write-layout.png" alt="Fig. 8. SRAM write circuit layout." width="600"><br>
<em>Fig. 8. SRAM write circuit layout.</em>
</div>

<div align="center">
<img src="figures/report/fig11-sram-read-layout.png" alt="Fig. 9. SRAM read circuit layout." width="600"><br>
<em>Fig. 9. SRAM read circuit layout.</em>
</div>

<div align="center">
<img src="figures/report/fig12-sram-array-layout.png" alt="Fig. 10. 8x8 SRAM array layout." width="460"><br>
<em>Fig. 10. 8x8 SRAM array layout.</em>
</div>

### D. Datapath Layout

The datapath includes the adder/subtractor, shifter, mux, and accumulator latch. The arithmetic block generates the 8-bit result plus carry and overflow flags. The shifter and mux route the selected value into the accumulator latch.

<div align="center">
<img src="figures/report/fig13-adder-layout.png" alt="Fig. 11. Adder/subtractor layout." width="620"><br>
<em>Fig. 11. Adder/subtractor layout.</em>
</div>

<div align="center">
<img src="figures/report/fig14-shifter-layout.png" alt="Fig. 12. Shifter layout." width="700"><br>
<em>Fig. 12. Shifter layout.</em>
</div>

<div align="center">
<img src="figures/report/fig15-mux-layout.png" alt="Fig. 13. Mux layout." width="180"><br>
<em>Fig. 13. Mux layout.</em>
</div>

<div align="center">
<img src="figures/report/fig16-latch-layout.png" alt="Fig. 14. Accumulator latch layout." width="260"><br>
<em>Fig. 14. Accumulator latch layout.</em>
</div>

## IV. Physical Verification

The completed top-level layout passes DRC and LVS. The DRC result reports no rule violations, and LVS reports a successful comparison between the extracted layout and schematic netlists.

| Check | Result |
| --- | --- |
| DRC | Passed, no results found |
| LVS | Passed, extracted and schematic netlists match |
| Final layout | Included in `Layout_files/ps9_Microprocessor.gds` |

<div align="center">
<img src="figures/report/fig02-drc-result.png" alt="Fig. 15. DRC result." width="420"><br>
<em>Fig. 15. DRC result.</em>
</div>

<div align="center">
<img src="figures/report/fig03-lvs-result.png" alt="Fig. 16. LVS result." width="420"><br>
<em>Fig. 16. LVS result.</em>
</div>

## V. Functional Verification

### A. Test Operation Table

The transient test sequence loads memory with known values, executes accumulator and arithmetic operations, verifies shift behavior, and stores the final memory values to the external bus.

| Step | Operation | `INSTR` | Memory Address | Binary Value | Decimal Value | `C` | `OV` | `SHIFT<2:0>` |
| ---: | --- | --- | ---: | --- | ---: | ---: | ---: | --- |
| 1 | `LOAD` | `001b` | 0 | `00000000b` | 0 |  |  |  |
| 2 | `LOAD` | `001b` | 1 | `01001001b` | 73 |  |  |  |
| 3 | `LOAD` | `001b` | 2 | `10010010b` | -110 |  |  |  |
| 4 | `LOAD` | `001b` | 3 | `11011011b` | -37 |  |  |  |
| 5 | `LOAD` | `001b` | 4 | `00100100b` | 36 |  |  |  |
| 6 | `LOAD` | `001b` | 5 | `01101101b` | 109 |  |  |  |
| 7 | `LOAD` | `001b` | 6 | `10110110b` | -74 |  |  |  |
| 8 | `LOAD` | `001b` | 7 | `11111111b` | -1 |  |  |  |
| 9 | `NOP` | `000b` | 0 |  |  |  |  |  |
| 10 | `GET` | `011b` | 3 | `11011011b` | -37 |  |  |  |
| 11 | `ADD` | `101b` | 4 | `00100100b` | 36 | 0 | 0 |  |
| 12 | `PUT` | `100b` | 0 | `11111111b` | -1 |  |  |  |
| 13 | `NOP` | `000b` | 0 |  |  |  |  |  |
| 14 | `GET` | `011b` | 1 | `01001001b` | 73 |  |  |  |
| 15 | `ADD` | `101b` | 5 | `01101101b` | 109 | 0 | 1 |  |
| 16 | `PUT` | `100b` | 1 | `10110110b` | -74 |  |  |  |
| 17 | `NOP` | `000b` | 0 |  |  |  |  |  |
| 18 | `GET` | `011b` | 5 | `01101101b` | 109 |  |  |  |
| 19 | `SUB` | `110b` | 4 | `00100100b` | 36 | 1 | 0 |  |
| 20 | `PUT` | `100b` | 2 | `01001001b` | 73 |  |  |  |
| 21 | `NOP` | `000b` | 0 |  |  |  |  |  |
| 22 | `GET` | `011b` | 3 | `11011011b` | -37 |  |  |  |
| 23 | `SUB` | `110b` | 5 | `01101101b` | 109 | 1 | 1 |  |
| 24 | `PUT` | `100b` | 3 | `01101110b` | 110 |  |  |  |
| 25 | `NOP` | `000b` | 0 |  |  |  |  |  |
| 26 | `GET` | `011b` | 0 | `11111111b` | -1 |  |  |  |
| 27 | `SHIFT` | `111b` | 0 |  |  |  |  | `011` |
| 28 | `PUT` | `100b` | 4 | `11111000b` | -8 |  |  |  |
| 29 | `NOP` | `000b` | 0 |  |  |  |  |  |
| 30 | `GET` | `011b` | 3 | `01101110b` | 110 |  |  |  |
| 31 | `SHIFT` | `111b` | 0 |  |  |  |  | `101` |
| 32 | `PUT` | `100b` | 5 | `11000000b` | -64 |  |  |  |
| 33 | `STORE` | `010b` | 0 | `11111111b` | -1 |  |  |  |
| 34 | `STORE` | `010b` | 1 | `10110110b` | -74 |  |  |  |
| 35 | `STORE` | `010b` | 2 | `01001001b` | 73 |  |  |  |
| 36 | `STORE` | `010b` | 3 | `01101110b` | 110 |  |  |  |
| 37 | `STORE` | `010b` | 4 | `11111000b` | -8 |  |  |  |
| 38 | `STORE` | `010b` | 5 | `11000000b` | -64 |  |  |  |
| 39 | `STORE` | `010b` | 6 | `10110110b` | -74 |  |  |  |
| 40 | `STORE` | `010b` | 7 | `11111111b` | -1 |  |  |  |

### B. Waveform Results

The instruction waveform verifies the applied opcode and address fields. The external bus waveform verifies that memory load values are accepted and that store operations reproduce the expected output sequence.

<div align="center">
<img src="figures/report/fig17-waveforms-instruction-bus.png" alt="Fig. 17. Instruction and external bus waveform verification." width="760"><br>
<em>Fig. 17. Instruction and external bus waveform verification.</em>
</div>

The status and shift waveform verifies `SHIFT_BYPASS`, carry, overflow, and shift control behavior. The delay plot compares `PHI1` and `EXT_BUS<0>` around a representative output transition.

<div align="center">
<img src="figures/report/fig18-waveforms-shift-delay.png" alt="Fig. 18. Shift, carry, overflow, and delay waveform verification." width="760"><br>
<em>Fig. 18. Shift, carry, overflow, and delay waveform verification.</em>
</div>

### C. Delay Measurement

A representative timing measurement compares `PHI1` and `EXT_BUS<0>` at the 500 mV crossing. From the plotted cursor values, the output transition follows the clock transition by approximately 15 ps.

```text
PHI1 reference crossing ~= 40.080 ns
EXT_BUS<0> crossing     ~= 40.095 ns
Measured delay          ~= 15 ps
```

## VI. Design File Package

| File or Directory | Purpose |
| --- | --- |
| `EECS4321_Submission/eecs4321_submission.pdf` | Original project report source |
| `EECS4321_Submission/project_requirements.pdf` | Additional reference PDF |
| `Layout_files/ps9_Microprocessor.gds` | Final microprocessor layout database |
| `figures/report/` | Layout, DRC/LVS, and waveform figures used by this report |
| `VLSI_Project_Report.md` | DAC-style Markdown version of the microprocessor report |

## VII. Conclusion

An 8-bit custom CMOS microprocessor was implemented and verified at the layout level. The design integrates instruction decoding, control latching, SRAM, arithmetic, shifting, accumulator storage, and external bus transfer logic. The included layout figures document the completed physical blocks, while the DRC and LVS captures confirm physical-rule correctness and schematic-layout equivalence.

Functional transient simulation verifies the instruction sequence, memory loading, arithmetic operations, shift control, bus store behavior, carry and overflow outputs, and representative output timing. Schematic figures are omitted from this version and can be added later when the final schematic images are available.

## References

[1] Yi-Hsiang Wei and Zijian Shang, "Ps9 Microprocessor," project report PDF, 2025.
