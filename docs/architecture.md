# Architecture

## 1. Functional Specification

The perceptron computes:

```
Yout = X1*W1 + X2*W2
```

- `X1`, `X2`: 4-bit signed inputs (two's complement), toggling at 1 GHz.
- `W1`, `W2`: 2-bit weight inputs, treated as external control signals (weight memory is out of scope), selecting a multiplication factor via right-shift:

| W (S1 S0) | Shift amount | Multiplication factor |
|---|---|---|
| 00 | 0 bits | ×1 |
| 01 | 1 bit  | ×0.5 |
| 10 | 2 bits | ×0.25 |
| 11 | 3 bits | ×0.125 |

- `Yout`: 5-bit signed output (two's complement), sized to avoid overflow from summing two 4-bit signed products.

Because right shift implements floor division, results for negative multiplication factors round down (toward negative infinity), which is expected and documented behavior, not a bug.

## 2. Building Blocks

### 2.1 Standard Cells
Custom CMOS logic gates were designed at the transistor level: inverter (NOT), NAND, NOR, AND (NAND+INV), OR (NOR+INV), and XOR (NAND/NOR combination). These are the base cells feeding into every higher-level block, and their individual delay characterization is the foundation of the stage-by-stage delay analysis (see `delay-analysis.md`).

### 2.2 2:1 Multiplexer (MUX)
Built from a transmission-gate topology (parallel NMOS/PMOS pair) rather than gate-level logic, for minimal propagation delay and near-ideal voltage transfer. Input buffers restore signal levels before the transmission gates; an output inverter restores drive strength afterward.

### 2.3 4-bit Barrel Shifter (Multiplier)
Two cascaded stages of 2:1 MUXes:
- **Stage 1** (controlled by `S0`): shift by 0 or 1 bit.
- **Stage 2** (controlled by `S1`): shift by 0 or 2 bits.

The sign bit (`A3`) is always propagated into vacated MSB positions on right shift, preserving correct two's-complement polarity for negative operands.

### 2.4 4-bit Carry Lookahead Adder (CLA)
Chosen over a ripple-carry adder specifically to meet the 1 GHz timing budget: ripple-carry delay grows linearly with bit width (worst-case chain through all carries), while CLA carry computation is parallelized.

- Each bit position computes `Generate (G = A·B)` and `Propagate (P = A⊕B)`.
- All carries `C1..C4` are computed directly from `C0`, `G[3:0]`, `P[3:0]` using the standard lookahead equations (no rippling).
- Sum bits: `Si = Pi ⊕ Ci`.
- A dedicated XOR gate (`P3 ⊕ C4`) produces the 5th (sign/overflow) output bit, correctly extending the signed 4-bit sum into 5-bit signed representation.

### 2.5 Top-Level Perceptron
Two barrel shifters run **in parallel** on `X1`/`W1` and `X2`/`W2`, producing `P1` and `P2`. These feed a single 4-bit CLA adder, producing the final 5-bit signed `Yout`. Because the two multiplications are independent and parallel, the critical path of the whole design is `t(one barrel shifter) + t(CLA adder)`, not `2 × t(barrel shifter) + t(adder)`.

## 3. Verification

The design was verified in Cadence Virtuoso transient simulation:
- Barrel shifter in isolation, across random 4-bit signed inputs and all 4 weight codes, at 1 GHz.
- CLA adder in isolation, across random 4-bit signed operand pairs and carry-in, at 1 GHz.
- Full perceptron, integrating both barrel shifters and the CLA adder, across all 16 `(W1, W2)` combinations and random `X1`, `X2`, at 1 GHz — confirming stable, correct 5-bit signed output every clock cycle.
