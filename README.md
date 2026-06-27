# Microprocessor Core

This repository contains the report, layout database, verification figures, and submission files for an 8-bit custom CMOS microprocessor project.

## Repository Layout

| Path | Description |
| --- | --- |
| `VLSI_Microprocessor_Report.md` | Main Markdown project report |
| `figures/report/` | Report-ready layout, DRC/LVS, and waveform figures |
| `Layout_files/ps9_Microprocessor.gds` | Final microprocessor GDS layout database |
| `EECS4321_Submission/eecs4321_submission.pdf` | Final submission PDF |
| `EECS4321_Submission/project_requirements.pdf` | Project requirements/reference PDF |

## Report

Open the Markdown report here:

[VLSI_Microprocessor_Report.md](VLSI_Microprocessor_Report.md)

The report currently includes only layout, DRC, LVS, and waveform figures. Schematic figures are intentionally omitted and can be added later.

## Design Summary

The microprocessor is an 8-bit custom CMOS design with:

- PLA-based instruction decoding
- Control-signal latch
- 8x8 SRAM
- Adder/subtractor datapath
- Shifter and mux
- Accumulator latch
- Bidirectional external bus driver
- DRC/LVS verification
- Transient waveform verification

## Instruction Set

| Opcode | Instruction | Function |
| --- | --- | --- |
| `000` | `NOP` | No operation |
| `001` | `LOAD` | `Mem[i] <- External Bus` |
| `010` | `STORE` | `External Bus <- Mem[i]` |
| `011` | `GET` | `Acc <- Mem[i]` |
| `100` | `PUT` | `Mem[i] <- Acc` |
| `101` | `ADD` | `Acc <- Acc + Mem[i]` |
| `110` | `SUB` | `Acc <- Acc - Mem[i]` |
| `111` | `SHIFT` | Shift accumulator left by `i` |
