# Propagation Delay Analysis

All hand calculations use the standard first-order Elmore delay model:

```
tp = 0.69 * Req * Ceq
```

## 1. Device / Technology Parameters

| Parameter | NMOS | PMOS |
|---|---|---|
| Req | 6.4148 kΩ | 6.0852 kΩ |
| Cd (drain/diffusion cap) | 0.2803 fF | 0.5413 fF |
| Cg (gate cap) | 0.1203 fF | 0.2244 fF |

Derived combined values:
- `Cgn + Cgp = 0.3447 fF`
- `Cdn + Cdp = 0.8216 fF`
- Average equivalent resistance used across gates: `Ravg = R = 6.25 kΩ`
- External load capacitance: `Cl = 1 fF`

## 2. Standard Gate Delays (Hand vs. Simulation)

| Gate | tp,hand | tp,sim (avg of LH/HL) | Error |
|---|---|---|---|
| Inverter | 7.856 ps (total, incl. Cl) / 3.543 ps (intrinsic) | 3.132 ps | 13.14% |
| NAND | 11.399 ps (total) / 7.086 ps (intrinsic) | 11.084 ps | 2.84% |
| AND | 16.428 ps (total) / 12.116 ps (intrinsic) | 16.548 ps | 0.73% |
| OR | 16.428 ps (total) / 12.116 ps (intrinsic) | 16.019 ps | 2.56% |
| XOR | 25.000 ps (total) / 20.688 ps (intrinsic) | 23.544 ps | 6.19% |

These per-gate figures (`tp,int`, intrinsic delay excluding the final external load) are the building blocks used in the multi-stage delay chains below.

## 3. Multiplier (Barrel Shifter) Delay

Two-stage cascade of transmission-gate 2:1 MUXes (inverter buffer + transmission gate + inverter buffer, per stage).

- **Stage 1** delay: `tp1 = 17.889 ps` (buffer-in + TG + buffer-out chain)
- **Stage 2** delay: `tp2 = 21.38 ps` (adds the load presented by the following CLA's AND/XOR inputs)
- **Total multiplier delay:** `t_multi = tp1 + tp2 ≈ 40.22 ps` (39.269 ps using the refined LH/HL average)

| | Hand | Simulation | Error |
|---|---|---|---|
| Multiplier delay | 39.269 ps | 46.91 ps | 16.29% |

## 4. CLA Adder Delay (Critical Path)

Critical path was isolated for the transition `X2<2>: 0→1` (with `X1 = 1111`, `W1 = W2 = 00`), tracing all logic stages contributing real delay from the barrel-shifter outputs to `SUM<4>`. Several intermediate lookahead terms evaluate to a constant 0 for this vector (e.g. `K3`, `out3`, `K2`, `K4`, `out7`) and were excluded, since a constant signal contributes no switching delay.

| Stage | Gate | Description | Delay |
|---|---|---|---|
| 1 | AND | `G<2>` from `B1<2>, B2<2>` | 16.57 ps |
| 2 | XOR | `P<2>` from `B1<2>, B2<2>` | 31.16 ps |
| 3 | AND | `out1` from `P<3>, G<2>` | 14.57 ps |
| 4 | OR  | `out2` from `out1, G<3>` | 14.57 ps |
| 5 | OR  | `out4` from `out3, out2` | 14.57 ps |
| 6 | OR  | `out6` from `out4, out5` | 14.57 ps |
| 7 | OR  | `C4` from `out7, out6` | 16.57 ps |
| 8 | XOR | `SUM<4>` from `P<3>, C4` (with `Cl = 1 fF`) | 25.00 ps |

**Total CLA adder delay (hand):** `116.42 ps`

| | Hand | Simulation | Error |
|---|---|---|---|
| CLA adder delay | 116.42 ps | 118.9 ps | 2.09% |

## 5. Full Perceptron Delay

```
t_perceptron = t_multiplier + t_adder = 39.269 ps + 116.42 ps = 155.689 ps
```

| | Hand | Simulation | Error |
|---|---|---|---|
| Full perceptron delay | 155.689 ps | 165.8 ps | 6.10% |

## 6. Discussion

- Errors between hand analysis and simulation stay within ~2–16% across all blocks, which is expected: the hand model uses a simplified, single average `Req` per device type and ignores second-order effects (velocity saturation, exact switching-threshold-dependent resistance, Miller effects on internal nodes) that Cadence's BSIM-level transistor models capture.
- The multiplier shows the largest relative error (16.29%), attributable to the transmission-gate stages, where the hand model approximates the parallel NMOS/PMOS pass-transistor resistance rather than modeling its bias-dependent, non-linear behavior.
- The CLA adder shows the best agreement (2.09%), since its critical path is dominated by simple static CMOS gates (AND/OR/XOR), which are well approximated by the Elmore RC model.
- See `layout-and-pex.md` for how these delays degrade once real interconnect parasitics (post-layout extraction) are included.
