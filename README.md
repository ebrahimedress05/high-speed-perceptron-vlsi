# High-Speed Perceptron — Transistor-Level Design (Cadence Virtuoso)

Transistor-level implementation of a signed 1 GHz artificial neuron (perceptron) built entirely from custom CMOS gates in Cadence Virtuoso (TSMC 65nm). The design computes `Yout = X1*W1 + X2*W2` for two 4-bit signed (two's complement) inputs, using arithmetic barrel shifters as simplified multipliers and a 4-bit Carry Lookahead Adder (CLA) for the accumulation stage.

This repository documents the architecture, and — most importantly — the **power** and **propagation delay** analysis of the design, comparing hand (first-order RC) calculations against Cadence Virtuoso transient/ADE simulation results, including post-layout (parasitic-extracted) numbers for the multiplier.

## Highlights

- 4-bit signed inputs `X1`, `X2`, 2-bit weight-controlled arithmetic right-shift multipliers (barrel shifters), 4-bit signed CLA adder, 5-bit signed output.
- Verified functionally at 1 GHz across all 16 weight combinations and both positive/negative operands.
- Full delay breakdown (gate-by-gate) for the multiplier and the CLA adder, hand-calculated and cross-checked against Cadence simulation.
- Power analysis (dynamic switching power) for the multiplier, hand-calculated vs. simulated.

## Repository Structure

```
.
├── README.md
├── docs/
│   ├── architecture.md        # System overview: barrel shifter, CLA, top-level perceptron
│   ├── delay-analysis.md      # Gate-level, multiplier, CLA adder and full-perceptron delay (hand vs. sim)
│   └── power-analysis.md      # Dynamic power hand analysis vs. Cadence ADE measurement
├── results/
│   └── summary.md             # Consolidated numeric results table (delay + power, hand vs. sim, error %)
└── cadence/
    ├── README.md               # How to open/reattach the PDK and continue work in Virtuoso
    ├── cds.lib                 # Library definition file
    └── PROJECT/                # Cadence Virtuoso OA library (schematics, symbols, testbenches)
```

The `cadence/PROJECT/` folder is the actual working design database (schematics, symbols, and ADE-XL testbench setups for every cell, from the base gates up to the full perceptron) so the project can be reopened and continued in Virtuoso, not just read about. See [`cadence/README.md`](cadence/README.md) for setup instructions and a note on the (proprietary, not included) PDK dependency.

## Design Summary

| Parameter | Value |
|---|---|
| Technology | TSMC 65nm |
| Clock / input rate | 1 GHz (1 ns period) |
| Input format | 4-bit signed, two's complement |
| Weight format | 2-bit (×1, ×0.5, ×0.25, ×0.125 via right shift) |
| Output | 5-bit signed, two's complement |
| Multiplier implementation | 4-bit arithmetic barrel shifter (two cascaded transmission-gate 2:1 MUX stages) |
| Adder implementation | 4-bit Carry Lookahead Adder (CLA) with signed overflow/sign-bit generation |

See [`docs/architecture.md`](docs/architecture.md) for the full block-level description.

## Key Results (see `results/summary.md` for full tables)

| Metric | Hand Analysis | Cadence Simulation | Error |
|---|---|---|---|
| Multiplier delay | 39.269 ps | 46.91 ps | 16.29% |
| CLA adder delay | 116.42 ps | 118.9 ps | 2.09% |
| Full perceptron delay | 155.689 ps | 165.8 ps | 6.10% |
| Multiplier dynamic power | 10.25 µW | 11.33 µW | 9.53% |

## Notes

- All hand calculations use a first-order Elmore RC delay model (`tp = 0.69 * Req * Ceq`) with device parameters extracted per-transistor from the technology library.
- Power figures reflect purely dynamic switching power; simulation waveforms confirm zero static (leakage) power draw between transitions, consistent with ideal CMOS behavior.
- Full derivations, per-gate equations (NAND, NOR, AND, OR, XOR, inverter, 2:1 MUX), and stage-by-stage breakdowns are provided in the `docs/` folder.
