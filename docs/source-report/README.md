# Source Report Excerpt

`digital-design-power-delay-analysis.pdf` is an excerpt of the original project report, trimmed to the digital-design, delay, and power content that the rest of this repository (`docs/architecture.md`, `docs/delay-analysis.md`, `docs/power-analysis.md`) summarizes and cites.

## What's included (original report sections 1-7)

1. Introduction
2. Building Blocks - Logic Gates (inverter, NAND, NOR, AND, OR, XOR - transistor-level schematics and truth tables)
3. Building Blocks - Circuit Design (2:1 MUX, 4-bit barrel shifter/multiplier, CLA full adder, full perceptron integration)
4. Simulation Results and Functional Verification
5. Hand Analysis & Simulation Results for Logic Gate Delay
6. Hand Analysis & Simulation Results for Perceptron Delay
7. Hand-Analyzed and Simulated Power Consumption Results for the Multiplier

## What's excluded

- Cover page, abstract, and table of contents.
- Section 8, Layout Implementation Strategy (bonus physical-design section), and everything after it - out of scope for this repository.

## Why this file exists

The full original report also contains layout/PEX content and cover/administrative pages that are out of scope here. This excerpt keeps only the material that documents the actual digital design and its delay/power characterization, so the source and the summarized `docs/` pages stay traceable to each other.
