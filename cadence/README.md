# Cadence Virtuoso Library — `PROJECT`

This is the actual Cadence Virtuoso OpenAccess (OA) design library used to build and simulate the perceptron, included so the design can be reopened and continued in Virtuoso rather than just read about.

## Contents

`PROJECT/` is a standard Cadence cellview library. Each top-level folder is a cell, generally containing a `schematic/` view (and, where applicable, `symbol/` and `constraint/` views). Testbench cells also include an `adexl/` view holding the ADE-XL setup (`data.sdb`).

| Cell | Type | Description |
|---|---|---|
| `Inverter`, `Nand`, `Nor`, `And`, `Or`, `Xor` | Standard cell | Transistor-level CMOS gates |
| `Mux` | Standard cell | 2:1 transmission-gate multiplexer |
| `Multiplier` | Block | 4-bit arithmetic barrel shifter |
| `CLA_bit`, `CLA_logic`, `CLA` | Block | 1-bit CLA cell, carry-lookahead logic, full 4-bit CLA adder |
| `Perceptron` | Top | Full perceptron (2x Multiplier + CLA) |
| `inv_test`, `MuX_test`, `Multiplier_test`, `CLA_test`, `Perceptron_test`, `Perceptron_delay` | Testbench | ADE-XL transient/delay testbenches for the corresponding cell |

## What was intentionally excluded

To keep the repository clean and portable, the following were stripped from the original Cadence work area before committing:
- Session lock files (`*.cdslck*`) - machine/session-specific, regenerated automatically by Virtuoso.
- Run logs (`*.log`) and result databases (`*.rdb`) - reproducible simulation outputs, not source data.
- `sevSaveDir/` folders and `adexl/test_states/` - ADE-XL run-history snapshots, regenerated on each simulation run.
- Auto-generated schematic thumbnail previews (`thumbnail_*.png`).

What remains is exactly what is needed to reopen every schematic, symbol, and testbench setup (`data.sdb`) in Virtuoso and continue working - the design database itself.

## How to Open This Project

1. Copy (or symlink) the `cadence/` folder into your Cadence work area.
2. Make sure `cadence/cds.lib` is picked up - either launch `virtuoso` from inside `cadence/`, or add `INCLUDE <path-to>/cadence/cds.lib` to your own top-level `cds.lib`.
3. **Technology / PDK:** This library was built against the TSMC 65nm technology (as referenced by the device models used in the schematics, e.g. `tsmcN65 nch_mac` / `tsmcN65 pch_mac`). The PDK itself is proprietary and is **not** included in this repository - you will need access to it separately (e.g. through your institution) and must attach it in your own `cds.lib` before the schematics will resolve devices and simulate correctly.
4. Open the `Perceptron` cell's `schematic` view as the top-level entry point, or open `Perceptron_test` / `Perceptron_delay` directly to rerun the transient/delay testbenches in ADE-XL.

## Notes for Continuing the Work

- Testbenches (`*_test`, `Perceptron_delay`) carry their ADE-XL setup (`adexl/data.sdb`) but not prior run results - running them fresh in Virtuoso will regenerate waveforms and logs locally.
- See `docs/architecture.md`, `docs/delay-analysis.md`, and `docs/power-analysis.md` in the repository root for the design rationale and the numeric analysis this library was used to produce.
