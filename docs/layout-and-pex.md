# Layout Implementation & Post-Layout (PEX) Delay Impact — Bonus

## 1. Scope

Per the bonus task, only the **multiplier** (4-bit barrel shifter) was taken to physical layout in Cadence Virtuoso; the adder remained schematic-only, as permitted by the assignment.

## 2. Layout Methodology (Bottom-Up)

1. **Phase 1 — Fundamental cell layout:** Custom layout of the CMOS inverter, verified DRC-clean and LVS-clean against its schematic.
2. **Phase 2 — Sub-block integration:** The verified inverter layout (plus other required basic gates) was instantiated to build the 2:1 MUX layout, routed and optimized for area/delay, and verified DRC/LVS-clean.
3. **Phase 3 — Top-level integration:** Multiple MUX instances were integrated and routed to form the complete multiplier layout. DRC passed clean (only density-rule flags remain, which are only meaningful in a full-chip context, not at the block level). LVS passed clean against the multiplier schematic.

## 3. Post-Layout Simulation Flow

1. **Parasitic Extraction (PEX):** Performed on the verified multiplier layout, extracting real interconnect and device-proximity parasitic R and C.
2. **Top-level substitution:** The PEX-extracted view of the multiplier replaced its ideal schematic view inside the full perceptron testbench, enabling a realistic, parasitics-included transient simulation of the complete design.

## 4. Delay Degradation Results

| Case | Pre-layout (schematic) delay | Post-layout (PEX) delay | Degradation |
|---|---|---|---|
| Worst case | ~160 ps | 289.291 ps (+123.491 ps) | ~74% |
| Best case | ~76.59 ps | 86.53 ps (+9.939 ps) | ~13% |

## 5. Discussion

- The wide spread between best-case (~13%) and worst-case (~74%) degradation reflects how strongly the physical routing path length and node loading depend on which specific input transition is exercised — paths through more metal interconnect and more device-proximity coupling pick up proportionally more parasitic RC delay.
- This confirms the expected trend in physical design: post-layout parasitics always increase delay relative to the ideal schematic, and this must be accounted for when closing timing at 1 GHz — schematic-only delay numbers are optimistic and should not be used as the final sign-off figure.
- These results validate that the layout extraction flow (DRC → LVS → PEX → post-layout simulation) was executed correctly and is consistent with standard VLSI physical design expectations.
