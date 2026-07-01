# Full-Custom 8-Bit Microprocessor Core in 65 nm CMOS

Yi-Hsiang Wei and Zijian Shang  
Department of Electrical Engineering, Columbia University  
VLSI Design Project

## Abstract

This report presents the layout, physical verification, and waveform-level functional verification of a full-custom 8-bit microprocessor core in 65 nm CMOS. The processor integrates a PLA-based instruction decoder, a control-signal latch, an 8-by-8 SRAM, an arithmetic datapath, a shifter, an accumulator latch, a multiplexer, and a bidirectional external bus interface.

The instruction set supports eight operations: `NOP`, `LOAD`, `STORE`, `GET`, `PUT`, `ADD`, `SUB`, and `SHIFT`. The included figures document the top-level schematic, layout views, DRC/LVS verification captures, and transient waveform results.

## I. Introduction

The goal of this project is to design and verify a compact full-custom 8-bit microprocessor core. The processor operates on 8-bit data words and uses a 3-bit opcode and memory address bits to select memory, arithmetic, shift, and external bus operations.

The major top-level signals are summarized below.

| Signal | Direction | Description |
| --- | --- | --- |
| `PHI1`, `PHI2` | Input | Two-phase clock signals |
| `INSTR<0:5>` | Input | Instruction code containing opcode and memory address fields |
| `EXT_BUS<0:7>` | Bidirectional | External 8-bit data bus |
| `SHIFT_BYPASS` | Control | Selects shift-bypass mode |
| `C` | Output | Carry flag |
| `OV` | Output | Overflow flag |

<div align="center"><strong>Table 1. Top-Level Signal Summary</strong></div>

<br>

## II. Architecture

### A. Top-Level Datapath

The top-level schematic and layout show the complete processor hierarchy. The datapath connects the SRAM, accumulator latch, arithmetic unit, shifter, multiplexer, and external bus driver. The control path decodes `INSTR<0:2>` and latches timing-sensitive control signals for datapath evaluation.

<div align="center">
<img src="figures/fig01-top-level-schematic.jpg" alt="Fig. 1. Top-level microprocessor schematic." width="1000"><br>
<em>Fig. 1. Top-level microprocessor schematic.</em>
</div>

<div align="center">
<img src="figures/fig02-top-level-layout.jpg" alt="Fig. 2. Top-level microprocessor layout." width="1000"><br>
<em>Fig. 2. Top-level microprocessor layout.</em>
</div>

<br>

The external bus can load memory during `LOAD` or receive stored data during `STORE`. Arithmetic operations use the accumulator and SRAM data as operands.

### B. Instruction Set and Control Table

Table 2 summarizes the opcode and decoded control behavior.

| Instruction | Opcode | Function | `SUB` | `MUX2 (SRAM)` | `MUX1 (Adder)` | `MUX0 (Shifter)` | `MEM_WRITE` | `MEM_READ` | `DRV_EN` | `SHIFT_BYPASS` | `LOAD_BUS` | `STORE_BUS` |
| --- | --- | --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| `NOP` | `000` | No&nbsp;operation | - | 0 | 0 | 1 | 0 | - | 0 | 0 | 0 | 0 |
| `LOAD` | `001` | Mem[`i`]&nbsp;&lt;-&nbsp;External&nbsp;Bus | - | - | - | - | 1 | - | 0 | - | 1 | 0 |
| `STORE` | `010` | External&nbsp;Bus&nbsp;&lt;-&nbsp;Mem[`i`] | - | 1 | 0 | 0 | 0 | 1 | 0 | - | 0 | 1 |
| `GET` | `011` | Acc&nbsp;&lt;-&nbsp;Mem[`i`] | - | 1 | 0 | 0 | 0 | 1 | 0 | - | 0 | 0 |
| `PUT` | `100` | Mem[`i`]&nbsp;&lt;-&nbsp;Acc | - | - | - | - | 1 | - | 1 | - | 0 | 0 |
| `ADD` | `101` | Acc&nbsp;&lt;-&nbsp;Acc&nbsp;+&nbsp;Mem[`i`] | 0 | 0 | 1 | 0 | 0 | 1 | 0 | 0 | 0 | 0 |
| `SUB` | `110` | Acc&nbsp;&lt;-&nbsp;Acc&nbsp;-&nbsp;Mem[`i`] | 1 | 0 | 1 | 0 | 0 | 1 | 0 | 0 | 0 | 0 |
| `SHIFT` | `111` | Shift&nbsp;accumulator&nbsp;left&nbsp;by&nbsp;`1` | - | 0 | 0 | 1 | 0 | - | 0 | 1 | 0 | 0 |

<div align="center"><strong>Table 2. Instruction Set and Control Signals</strong></div>

<br>

## III. Physical Verification

The completed top-level layout passes DRC and LVS. The DRC result reports no rule violations, and the LVS result confirms a successful comparison between the extracted layout and schematic netlists. The final layout database is included in `Layout_files/ps9_Microprocessor.gds`.

<table>
<tr>
<td align="center">
<img src="figures/fig03-drc-result.jpg" alt="Fig. 3. DRC result." width="500"><br>
<em>Fig. 3. DRC result.</em>
</td>
<td align="center">
<img src="figures/fig04-lvs-result.jpg" alt="Fig. 4. LVS result." width="500"><br>
<em>Fig. 4. LVS result.</em>
</td>
</tr>
</table>

<br>

## IV. Implementation

### A. Instruction-Decoder PLA

The instruction-decoder PLA is generated from the control behavior summarized in Table 2. The control table was translated into the Espresso input file `Espresso_files/instr_decoder.pla`, which defines three opcode inputs, `instr2`, `instr1`, and `instr0`, and ten decoded control outputs: `subtract`, `mux2`, `mux1`, `mux0`, `mem_write`, `mem_read`, `drv_enable`, `shift_bypass`, `load_bus`, and `store_bus`. Unused control values are marked as don't-care entries so Espresso can minimize the logic instead of forcing every output to a fixed value.

<div align="center">
<img src="figures/fig05-instr-decoder-pla.jpg" alt="Fig. 5. Instruction decoder PLA input file." width="800"><br>
<em>Fig. 5. Instruction decoder PLA input file.</em>
</div>

<br>

The input PLA file was then processed with Espresso to generate the minimized output file, `Espresso_files/instr_decoder_out.pla`. Espresso preserves the same input and output interface while replacing selected opcode-specific rows with shared implicants, such as `-00`, `0-1`, `-01`, and `1-0`. These minimized product terms reduce the amount of logic required by the instruction decoder.

<div align="center">
<img src="figures/fig06-instr-decoder-out-pla.jpg" alt="Fig. 6. Espresso-minimized instruction decoder PLA output file." width="800"><br>
<em>Fig. 6. Espresso-minimized instruction decoder PLA output file.</em>
</div>

<br>

The Espresso-minimized output was used to build the instruction-decoder PLA symbol, schematic, and layout. The symbol provides the block-level interface, the schematic maps the minimized product terms to the decoded control outputs, and the final layout implements the same logic in a regular row-and-column PLA structure.

<div align="center">
<img src="figures/fig07-pla-symbol.jpg" alt="Fig. 7. PLA symbol." width="1000"><br>
<em>Fig. 7. PLA symbol.</em>
</div>

<div align="center">
<img src="figures/fig08-pla-schematic.jpg" alt="Fig. 8. PLA schematic." width="1000"><br>
<em>Fig. 8. PLA schematic.</em>
</div>

| Cell | Device | Length | Width | Fingers |
| --- | --- | ---: | ---: | ---: |
| PLA AND plane | PMOS | 60 nm | 1 um | 1 |
|  | NMOS | 60 nm | 1 um | 1 |
| PLA OR plane | PMOS | 60 nm | 1 um | 1 |
|  | NMOS | 60 nm | 1 um | 1 |
| PLA inverter | PMOS | 60 nm | 1 um | 2 |
|  | NMOS | 60 nm | 1 um | 1 |

<div align="center"><strong>Table 3. PLA Cell Sizing</strong></div>

<br>

<div align="center">
<img src="figures/fig09-pla-layout.jpg" alt="Fig. 9. PLA layout." width="1000"><br>
<em>Fig. 9. PLA layout.</em>
</div>

<br>

### B. Control-Signal Latch

The control-signal latch stores selected PLA outputs so the datapath control signals remain stable during evaluation. The block latches the control bits used for subtraction, multiplexer selection, and shift control, then presents those stored signals to the datapath during the active clock phase. Fig. 10 shows the block-level symbol, and Fig. 11 and Fig. 12 show the schematic and layout implementation.

<div align="center">
<img src="figures/fig10-control-latch-symbol.jpg" alt="Fig. 10. Control-signal latch symbol." width="1000"><br>
<em>Fig. 10. Control-signal latch symbol.</em>
</div>

<div align="center">
<img src="figures/fig11-control-latch-schematic.jpg" alt="Fig. 11. Control-signal latch schematic." width="1000"><br>
<em>Fig. 11. Control-signal latch schematic.</em>
</div>

<div align="center">
<img src="figures/fig12-control-latch-layout.jpg" alt="Fig. 12. Control-signal latch layout." width="1000"><br>
<em>Fig. 12. Control-signal latch layout.</em>
</div>

<br>

The control-signal latch is built from one inverter cell and five latch cells. The inverter generates the complementary clock phase required by the latch stages, while each latch cell stores one decoded control bit from the PLA. The inverter uses a 2x PMOS finger count to balance the stronger NMOS pull-down path. The latch switch devices use matched 700 nm PMOS and NMOS widths so the pass path remains compact while providing balanced switching behavior. Fig. 13 through Fig. 16 show the reusable inverter and latch cells, and Table 4 lists the device sizes.

<table>
<tr>
<td align="center">
<img src="figures/fig13-control-latch-inverter-schematic.jpg" alt="Fig. 13. Inverter schematic used in the control latch." width="850"><br>
<em>Fig. 13. Inverter schematic used in the control latch.</em>
</td>
<td align="center">
<img src="figures/fig14-control-latch-inverter-layout.jpg" alt="Fig. 14. Inverter layout used in the control latch." width="150"><br>
<em>Fig. 14. Inverter layout used in the control latch.</em>
</td>
</tr>
</table>

<table>
<tr>
<td align="center">
<img src="figures/fig15-control-latch-latch-schematic.jpg" alt="Fig. 15. Latch schematic used in the control latch." width="550"><br>
<em>Fig. 15. Latch schematic used in the control latch.</em>
</td>
<td align="center">
<img src="figures/fig16-control-latch-latch-layout.jpg" alt="Fig. 16. Latch layout used in the control latch." width="450"><br>
<em>Fig. 16. Latch layout used in the control latch.</em>
</td>
</tr>
</table>

| Cell | Device | Length | Width | Fingers |
| --- | --- | ---: | ---: | ---: |
| Control-latch inverter | PMOS | 60 nm | 1 um | 2 |
|  | NMOS | 60 nm | 1 um | 1 |
| Control-latch switch | PMOS | 60 nm | 700 nm | 1 |
|  | NMOS | 60 nm | 700 nm | 1 |

<div align="center"><strong>Table 4. Control-Latch Cell Sizing</strong></div>

<br>

### C. SRAM Memory Design

The SRAM memory subsystem stores eight 8-bit words and provides the processor's data-memory interface. The block uses `PHI1`, `PHI2`, `MEM_READ`, `MEM_WRITE`, an 8-bit bidirectional data bus, and a 3-bit address input. It is organized into the top-level SRAM interface, the 8-by-8 core array, the row decoder, `PHI2` precharge circuitry, `PHI1`-qualified write circuitry, and the `MEM_READ`-controlled read path.

<div align="center">
<img src="figures/fig17-sram-symbol.jpg" alt="Fig. 17. SRAM symbol." width="500"><br>
<em>Fig. 17. SRAM symbol.</em>
</div>

<div align="center">
<img src="figures/fig18-sram-schematic.jpg" alt="Fig. 18. SRAM schematic." width="500"><br>
<em>Fig. 18. SRAM schematic.</em>
</div>

<div align="center">
<img src="figures/fig19-sram-layout.jpg" alt="Fig. 19. SRAM layout." width="1000"><br>
<em>Fig. 19. SRAM layout.</em>
</div>

<br>

#### SRAM Core Array

The SRAM core implements the required eight-word by eight-bit storage array. It is based on the TSMC SRAM layout from `arrayLib`, with the provided 8-by-4 array expanded to an 8-by-8 organization by adding SRAM column instances and extending the edge, corner, and well-strap structures. Fig. 20 and Fig. 21 show the core schematic and layout before the array is connected to the row decoder, precharge network, write circuitry, and read circuitry.

<table>
<tr>
<td align="center">
<img src="figures/fig20-sram-core-schematic.jpg" alt="Fig. 20. SRAM core schematic." width="650"><br>
<em>Fig. 20. SRAM core schematic.</em>
</td>
<td align="center">
<img src="figures/fig21-sram-core-layout.jpg" alt="Fig. 21. SRAM core layout." width="350"><br>
<em>Fig. 21. SRAM core layout.</em>
</td>
</tr>
</table>

<br>

#### Row Decoder

The row decoder converts the 3-bit SRAM address field, `INSTR<3:5>`, into eight row-select outputs, with only one SRAM row selected at a time. The inverters generate the complemented address bits required for decoding. First-stage three-input AND cells form the decoded row terms from the true and complemented address bits, and second-stage three-input AND cells qualify those row terms with `PHI1`. This structure enables the selected word line only during the active `PHI1` interval and keeps the decoder outputs disabled otherwise. Fig. 22 and Fig. 23 show the row-decoder schematic and layout.

<table>
<tr>
<td align="center">
<img src="figures/fig22-sram-decoder-schematic.jpg" alt="Fig. 22. SRAM decoder schematic." width="475"><br>
<em>Fig. 22. SRAM decoder schematic.</em>
</td>
<td align="center">
<img src="figures/fig23-sram-decoder-layout.jpg" alt="Fig. 23. SRAM decoder layout." width="525"><br>
<em>Fig. 23. SRAM decoder layout.</em>
</td>
</tr>
</table>

The decoder is implemented with three inverter cells and sixteen three-input AND cells. Each AND cell is built from a three-input NAND followed by an output inverter. The inverter uses a 2x PMOS finger count to balance the stronger NMOS pull-down path, and the NAND stage uses 3x NMOS fingers to compensate for the three-device series stack. Fig. 24 and Fig. 25 show these reusable decoder cells.

<div align="center">
<img src="figures/fig24-sram-decoder-inverter-schematic.jpg" alt="Fig. 24. SRAM decoder inverter schematic." width="500"><br>
<em>Fig. 24. SRAM decoder inverter schematic.</em>
</div>

<div align="center">
<img src="figures/fig25-sram-decoder-3input-and-schematic.jpg" alt="Fig. 25. SRAM decoder three-input AND schematic." width="500"><br>
<em>Fig. 25. SRAM decoder three-input AND schematic.</em>
</div>

| Cell | Device | Length | Width | Fingers |
| --- | --- | ---: | ---: | ---: |
| Row-decoder inverter | PMOS | 60 nm | 150 nm | 10 |
|  | NMOS | 60 nm | 150 nm | 5 |
| Row-decoder three-input AND | PMOS | 60 nm | 150 nm | 2 |
|  | NMOS | 60 nm | 150 nm | 3 |

<div align="center"><strong>Table 5. Row Decoder Cell Sizing</strong></div>

<br>

#### Precharge Circuits

The precharge circuit initializes the SRAM bitlines before row evaluation. During `PHI2`, the precharge devices pull both the bitline and bitline-bar nodes high so the selected cell can disturb the differential pair during the following `PHI1` interval. Fig. 26 and Fig. 27 show the full precharge schematic and layout.

<div align="center">
<img src="figures/fig26-sram-precharge-schematic.jpg" alt="Fig. 26. SRAM precharge schematic." width="1000"><br>
<em>Fig. 26. SRAM precharge schematic.</em>
</div>

<div align="center">
<img src="figures/fig27-sram-precharge-layout.jpg" alt="Fig. 27. SRAM precharge layout." width="1000"><br>
<em>Fig. 27. SRAM precharge layout.</em>
</div>

The one-bit precharge cell is the repeated column cell used for each SRAM bit. It uses two matched PMOS devices, one connected to the bitline and one connected to the bitline-bar node. Both devices use a 60 nm channel length, 300 nm width, and one finger so the complementary bitlines receive balanced pull-up strength during `PHI2`. Fig. 28 shows the one-bit precharge schematic, and Table 6 lists the device sizes.

<div align="center">
<img src="figures/fig28-sram-precharge-1bit-schematic.jpg" alt="Fig. 28. One-bit SRAM precharge schematic." width="500"><br>
<em>Fig. 28. One-bit SRAM precharge schematic.</em>
</div>

| Cell | Device | Length | Width | Fingers |
| --- | --- | ---: | ---: | ---: |
| Precharge cell | Bitline PMOS | 60 nm | 300 nm | 1 |
|  | Bitline-bar PMOS | 60 nm | 300 nm | 1 |

<div align="center"><strong>Table 6. Precharge Cell Sizing</strong></div>

<br>

#### Write Circuits

The write circuit drives data onto the selected SRAM column during memory write operations. The write path is qualified by `PHI1`, so the bitline drivers are active only during the evaluation phase when a selected word line is enabled. Fig. 29 and Fig. 30 show the full write schematic and layout.

<div align="center">
<img src="figures/fig29-sram-write-schematic.jpg" alt="Fig. 29. SRAM write schematic." width="1000"><br>
<em>Fig. 29. SRAM write schematic.</em>
</div>

<div align="center">
<img src="figures/fig30-sram-write-layout.jpg" alt="Fig. 30. SRAM write layout." width="1000"><br>
<em>Fig. 30. SRAM write layout.</em>
</div>

The one-bit write cell is the repeated driver used for each data bit. It takes the write data and its complement, then drives the bitline pair only when the write control path is enabled. This keeps unselected columns isolated and allows the same cell structure to be tiled across the 8-bit data path. Fig. 31 shows the one-bit write schematic.

<div align="center">
<img src="figures/fig31-sram-write-1bit-schematic.jpg" alt="Fig. 31. One-bit SRAM write schematic." width="500"><br>
<em>Fig. 31. One-bit SRAM write schematic.</em>
</div>

<br>

#### Read Circuits

The read circuit transfers the selected SRAM column value onto the internal data path during memory-read operations. The read driver is controlled by `MEM_READ`, which keeps the output path disabled when the memory is not being read. Fig. 32 and Fig. 33 show the complete read schematic and layout.

<div align="center">
<img src="figures/fig32-sram-read-schematic.jpg" alt="Fig. 32. SRAM read schematic." width="1000"><br>
<em>Fig. 32. SRAM read schematic.</em>
</div>

<div align="center">
<img src="figures/fig33-sram-read-layout.jpg" alt="Fig. 33. SRAM read layout." width="1000"><br>
<em>Fig. 33. SRAM read layout.</em>
</div>

The one-bit read cell is the repeated read path used for each SRAM data bit. It receives the selected bitline information and drives the output only when `MEM_READ` is asserted, preventing bus contention during non-read cycles. Fig. 34 shows the one-bit read schematic.

<div align="center">
<img src="figures/fig34-sram-read-1bit-schematic.jpg" alt="Fig. 34. One-bit SRAM read schematic." width="500"><br>
<em>Fig. 34. One-bit SRAM read schematic.</em>
</div>

<br>

### D. Datapath Block

The datapath includes the adder/subtractor, shifter, multiplexer, and accumulator latch. The arithmetic block generates the 8-bit result, carry flag, and overflow flag. The shifter and multiplexer route the selected value into the accumulator latch.

<div align="center">
<img src="figures/fig35-adder-layout.png" alt="Fig. 35. Adder/subtractor layout." width="620"><br>
<em>Fig. 35. Adder/subtractor layout.</em>
</div>

<div align="center">
<img src="figures/fig36-shifter-layout.png" alt="Fig. 36. Shifter layout." width="700"><br>
<em>Fig. 36. Shifter layout.</em>
</div>

<div align="center">
<img src="figures/fig37-mux-layout.png" alt="Fig. 37. Multiplexer layout." width="180"><br>
<em>Fig. 37. Multiplexer layout.</em>
</div>

<div align="center">
<img src="figures/fig38-latch-layout.png" alt="Fig. 38. Accumulator latch layout." width="260"><br>
<em>Fig. 38. Accumulator latch layout.</em>
</div>

<br>

## V. Functional Verification

### A. Test Operation Table

The transient test sequence loads memory with known values, executes accumulator and arithmetic operations, verifies shift behavior, and stores the final memory values on the external bus.

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

<div align="center"><strong>Table 7. Functional Verification Test Sequence</strong></div>

<br>

### B. Waveform Results

The instruction waveform verifies the applied opcode and address fields. The external bus waveform confirms that memory load values are accepted and that store operations reproduce the expected output sequence.

<div align="center">
<img src="figures/fig39-waveforms-instruction-bus.png" alt="Fig. 39. Instruction and external bus waveform verification." width="760"><br>
<em>Fig. 39. Instruction and external bus waveform verification.</em>
</div>

<br>

The status and shift waveforms verify `SHIFT_BYPASS`, carry, overflow, and shift-control behavior. The delay plot compares `PHI1` and `EXT_BUS<0>` around a representative output transition.

<div align="center">
<img src="figures/fig40-waveforms-shift-delay.png" alt="Fig. 40. Shift, carry, overflow, and delay waveform verification." width="760"><br>
<em>Fig. 40. Shift, carry, overflow, and delay waveform verification.</em>
</div>

<br>

### C. Delay Measurement

A representative timing measurement compares `PHI1` and `EXT_BUS<0>` at the 500 mV crossing. Based on the plotted cursor values, the output transition follows the clock transition by approximately 15 ps.

```text
PHI1 reference crossing ~= 40.080 ns
EXT_BUS<0> crossing     ~= 40.095 ns
Measured delay          ~= 15 ps
```

## VI. Design File Package

| File or Directory | Purpose |
| --- | --- |
| `EECS4321_Submission/eecs4321_submission.pdf` | Final submission report |
| `EECS4321_Submission/project_requirements.pdf` | Project requirements/reference PDF |
| `Layout_files/ps9_Microprocessor.gds` | Final microprocessor layout database |
| `figures/` | Schematic, layout, DRC/LVS, and waveform figures used by this report |
| `VLSI_Project_Report.md` | DAC-style Markdown version of the microprocessor report |

<div align="center"><strong>Table 8. Design File Package</strong></div>

<br>

## VII. Conclusion

A full-custom 8-bit microprocessor core in 65 nm CMOS was implemented and verified at the layout level. The design integrates instruction decoding, control latching, SRAM, arithmetic, shifting, accumulator storage, and external bus transfer logic. The included schematic and layout figures document the completed design hierarchy and physical blocks, while the DRC and LVS captures confirm design-rule correctness and schematic-layout equivalence.

Functional transient simulation verifies the instruction sequence, memory loading, arithmetic operations, shift control, bus store behavior, carry and overflow outputs, and representative output timing.

## References

[1] Yi-Hsiang Wei and Zijian Shang, "Ps9 Microprocessor," project report PDF, 2025.
