<!---

This file is used to generate your project datasheet. Please fill in the information below and delete any unused
sections.
-->

## How it works

A **4-Neuron Configurable 0-DSP Leaky Integrate-and-Fire (LIF) Array** designed for GF180 (`gf180mcuD`) framework verification without exposing proprietary weights or topologies:

- **Zero-Multiplier Leak (`ui_in[5:4]`)**:
  - `00`: No leak (`mem`)
  - `01`: `Beta = 0.5` (`mem >>> 1`)
  - `10`: `Beta = 0.25` (`mem >>> 2`)
  - `11`: `Beta = 0.125` (`mem >>> 3`)
- **4 Selectable Modes (`ui_in[7:6]`)**:
  - `00`: **Independent Integrators** — 4 separate LIF channels (`w = 32`, `threshold = 64`).
  - `01`: **Pairwise Spiking AND** — `(0&1)`, `(1&2)`, `(2&3)`, `(0&3)` with `w = 32`.
  - `10`: **Excitatory / Inhibitory Mix** — `+64` self-excitation and `-32` neighbor inhibition.
  - `11`: **4-Stage Spiking Cascade** — `spike_in[0] -> N0 -> N1 -> N2 -> N3` (`w = 64`).
- **24-Pin Full Observability**:
  - `uo_out[3:0]`: Live spikes from all 4 neurons.
  - `uo_out[7:4]`: Positive membrane potential indicators (`mem > 0`).
  - `uio_out[7:0]`: Live 8-bit signed membrane potential of Neuron 0.

## How to test

1. Apply reset (`rst_n = 0`, then `rst_n = 1`).
2. Select `shift_sel` on `ui_in[5:4]`, `mode` on `ui_in[7:6]`, and drive `ui_in[3:0]`.
3. Observe `uo_out[3:0]` for spikes and `uio_out[7:0]` for Neuron 0 membrane dynamics.

## External hardware

None required.
