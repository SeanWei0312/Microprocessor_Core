# Full-Custom 8-Bit Microprocessor Core in 65 nm CMOS

This project implements a full-custom 8-bit microprocessor core in 65 nm CMOS. The repository includes the final GDS layout, layout figures, DRC/LVS verification captures, waveform results, submission PDFs, and a Markdown project report.

## Project Summary

This is a VLSI design project for a compact 8-bit microprocessor core.

The microprocessor uses a PLA-based instruction decoder, a control-signal latch, an 8x8 SRAM, an adder/subtractor datapath, a shifter, a multiplexer, an accumulator latch, and a bidirectional external bus driver. The instruction set supports `NOP`, `LOAD`, `STORE`, `GET`, `PUT`, `ADD`, `SUB`, and `SHIFT`.

The current Markdown report focuses on layout implementation, DRC/LVS verification, and transient waveform verification. Schematic figures are intentionally omitted for now and can be added later.

## Contributors

Yi-Hsiang Wei and Zijian Shang are students in Columbia University's Department of Electrical Engineering.

- Yi-Hsiang Wei: full-custom CMOS schematic and layout implementation, Cadence Virtuoso design, and verification.
- Zijian Shang: report organization, result documentation, and project packaging.

## Key Results

| Metric | Result |
| --- | ---: |
| Data width | 8 bits |
| Memory size | 8x8 SRAM |
| Instruction opcode width | 3 bits |
| Supported instructions | 8 |
| Physical verification | DRC and LVS passed |
| Functional verification | Transient waveform tests passed |
| Measured output delay | approximately 15 ps |

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

## Files

- [VLSI_Project_Report.md](VLSI_Project_Report.md) - complete Markdown project report.
- [EECS4321_Submission/eecs4321_submission.pdf](EECS4321_Submission/eecs4321_submission.pdf) - final submission PDF.
- [EECS4321_Submission/project_requirements.pdf](EECS4321_Submission/project_requirements.pdf) - project requirements.
- `Layout_files/` - final GDS layout database.
- `figures/report/` - report-ready layout, DRC/LVS, and waveform figures.
- `EECS4321_Submission/` - organized submission package.

## Repository Layout

```text
Layout_files/
  ps9_Microprocessor.gds       Final microprocessor GDS layout database

figures/
  report/                      Layout, DRC/LVS, and waveform figures for the report

EECS4321_Submission/
  eecs4321_submission.pdf      Final submission report
  project_requirements.pdf     Project requirements/reference PDF

VLSI_Project_Report.md         Markdown project report
README.md                      Repository overview
```

## Report Notes

The report currently uses only layout, DRC, LVS, and waveform figures. When schematic figures are ready, add them to `figures/report/` and reference them from `VLSI_Project_Report.md` with relative paths such as:

```html
<img src="figures/report/figure-name.png" alt="Figure caption." width="700">
```
