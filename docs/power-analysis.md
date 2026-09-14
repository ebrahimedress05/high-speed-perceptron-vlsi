# Power Consumption Analysis (Multiplier)

## 1. Methodology

Power was analyzed for the 4-bit barrel shifter (multiplier) block using two complementary approaches:

1. **Simulation:** Cadence transient simulation with a pulse source (1 ns period) driving the inputs; the supply current spikes at each input transition were captured and used to compute average power as `P = I_avg * VDD`.
2. **Hand analysis:** Standard dynamic CMOS power model:

```
P = α * VDD^2 * C_total * f
```

where `α = 1` (a logic transition is assumed to occur every clock cycle — the worst-case/upper-bound switching activity factor).

## 2. Simulation Result

```
P_cadence = I_avg * VDD = 9.441666 µA * 1.2 V = 11.33 µW
```

The transient current waveforms confirm the expected CMOS behavior: current from VDD is essentially zero between transitions and spikes only during switching events, i.e. **zero static (leakage) power** in the ideal transistor models used — power is consumed exclusively through dynamic switching activity (charging/discharging of node capacitances).

## 3. Hand Analysis

Total switched capacitance for the multiplier's critical signal path was accumulated from the drain, gate, and standard-cell input capacitances along the switching nodes:

```
C_total = 2*Cdd + Cdd + 2*Cgg + Cdd + 2*Cgin,NAND + Cgin,NOR + 2*Cdd
        = 7.1181 fF
```

```
P_hand = α * VDD^2 * C_total * f
       = 1^2 * (1.2)^2 * 7.1181e-15 * 1e9
       = 10.25 µW
```

## 4. Comparison

| | Hand Analysis | Cadence Simulation | Error |
|---|---|---|---|
| Multiplier dynamic power | 10.25 µW | 11.33 µW | 9.53% |

## 5. Discussion

- The ~9.5% discrepancy is consistent with the delay-analysis errors and stems from the same source: the hand model lumps capacitances using nominal per-transistor values, while Cadence's simulation captures the actual, bias- and voltage-dependent charge delivered by VDD during each transition (including short-circuit current contributions from both NMOS and PMOS conducting briefly during switching, which the simple `CV²f` model does not explicitly separate out).
- Choosing `α = 1` is a conservative (worst-case) assumption appropriate for this design, since inputs change every clock cycle at 1 GHz with effectively random data — there is no idle/hold state where switching activity would be lower.
- No static power term appears in either the hand model or the simulation, confirming the design achieves ideal CMOS zero-static-power operation (aside from real-world leakage not captured at this level of modeling).
