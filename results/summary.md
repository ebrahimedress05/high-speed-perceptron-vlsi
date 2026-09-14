# Consolidated Results Summary

## Standard Gate Delays

| Gate | tp,hand (total) | tp,sim (avg LH/HL) | Error |
|---|---|---|---|
| Inverter | 7.856 ps | 3.132 ps | 13.14% |
| NAND (2-input) | 11.399 ps | 11.084 ps | 2.84% |
| AND (2-input) | 16.428 ps | 16.548 ps | 0.73% |
| OR (2-input) | 16.428 ps | 16.019 ps | 2.56% |
| XOR (2-input) | 25.000 ps | 23.544 ps | 6.19% |

## Block-Level Delay

| Block | Hand | Simulation | Error |
|---|---|---|---|
| Multiplier (barrel shifter) | 39.269 ps | 46.91 ps | 16.29% |
| CLA adder (critical path) | 116.42 ps | 118.9 ps | 2.09% |
| Full perceptron (multiplier + adder) | 155.689 ps | 165.8 ps | 6.10% |

## Power (Multiplier, dynamic)

| | Hand | Simulation | Error |
|---|---|---|---|
| Dynamic power @ 1 GHz | 10.25 µW | 11.33 µW | 9.53% |

## Post-Layout Delay Degradation (Multiplier, bonus)

| Case | Pre-layout | Post-layout (PEX) | Degradation |
|---|---|---|---|
| Worst case | ~160 ps | 289.291 ps | ~74% |
| Best case | ~76.59 ps | 86.53 ps | ~13% |

## Takeaways

- Hand-analysis vs. simulation error is under ~16% across every block, validating the first-order RC (Elmore) model as a reliable early-stage sizing/verification tool for this design.
- The Carry Lookahead Adder achieves its design goal: its 116–119 ps critical-path delay fits well within the 1 ns (1 GHz) clock period, with ample margin even after the multiplier's contribution is added (total ~156–166 ps).
- Physical layout parasitics are non-negligible and must be budgeted for: worst-case post-layout delay degradation reached ~74% on the multiplier alone, underscoring the importance of post-layout (PEX) verification before signing off a design at aggressive clock targets.
