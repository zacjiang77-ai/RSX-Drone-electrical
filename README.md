# RSX Jetson 19 V buck converter

This is a standalone daughterboard for the protected 6S branch of the RSX PDB. Open `RSX_Jetson_19V.kicad_pro` in KiCad 9 or later. Keep both project library tables and the project-local symbol and footprint libraries alongside the project.

| Parameter | Design value |
| --- | --- |
| Input at U1 under load | 19.8–25.2 V DC |
| Output | 19.0 V nominal |
| Jetson load assumption | 2.37 A, 45.0 W |
| Power-stage design target | 3.0 A, 57 W |
| Topology | LM70880RRXR synchronous buck |
| Switching frequency | 220 kHz nominal with 100 kΩ at RT |
| Board | 64 × 39 mm, four copper layers, four 3.2 mm mounting holes |

## Wiring

- J1 pin 1 takes protected 6S VBAT. J1 pin 2 is ground. Use the upstream protection and cutoff provided by the PDB. U1 EN follows the input and supplies no separate 6S cutoff.
- J2 pin 1 supplies +19 V to the center contact of the Jetson cable. J2 pin 2 goes to the barrel sleeve. Verify polarity at the actual cable before plugging in the Jetson.
- The board assumes the specified 19.8 V minimum at the regulator pins after cable and PDB losses.

## Design calculations

- Feedback setpoint is 0.8 V × (1 + 232 kΩ / 10.2 kΩ) = 18.996 V nominal.
- TI's off-time equation gives 19.59 V at 240 kHz and 126 ns. At 19.8 V there is approximately 0.21 V of margin before conduction losses, setpoint tolerance and ripple. In low-dropout mode the regulator can skip off cycles. Steady regulation at the 3 A target remains subject to bench validation.
- L1 is 33 µH nominal, with a specified maximum DCR of 20.7 mΩ at 25 °C. Ideal inductor ripple at 25.2 V is about 0.64 A peak-to-peak at 220 kHz and nominal inductance. U1 senses across the 10 mΩ shunt with separate Kelvin connections. The nominal cycle-by-cycle threshold is 5.6 A of peak inductor current.
- The Type II compensation network starts at 8.06 kΩ, 33 nF and 820 pF. It targets approximately 5.4 kHz crossover using TI's simplified calculation with 120 µF effective output capacitance. Actual DC bias, ESR and crossover need verification with the assembled output network.
- SW4 is internally connected to the switching node and is routed locally to the bootstrap capacitor. The main switch path uses SW1 through SW3.

## Layout and bring-up

1. Review the RRX0029B land pattern and stencil with the assembler. Specify filled and capped vias where they are inside solderable pads, including the ground thermal pad. Align L1's marked winding-start terminal with pad 1 toward U1.
2. Use a current-limited bench source and an electronic load. Start at 22.2 V and verify polarity, startup, and no-load output. Sweep input from 25.2 V to 19.8 V while checking 2.37 A and then the 3 A design target.
3. Scope VIN at U1 and VOUT at the Jetson cable during input and load steps. Check ripple, hot operation, loop response and recovery near dropout. Keep startup and unload peaks below the Jetson's 20 V DC jack maximum. Verify with the real battery, PDB and cable before flight.

KiCad 9.0.2 ERC, board DRC and schematic parity checks pass. Hardware testing is pending.

## References

- [TI LM70880 datasheet and RRX0029B package drawing](https://www.ti.com/lit/ds/symlink/lm70880.pdf)
- [Coilcraft XGL1313-333 inductor specifications and land pattern](https://www.coilcraft.com/en-us/products/power/high-voltage-inductors/xgl/xgl1313/xgl1313-333/)
- [NVIDIA Jetson Orin Nano developer kit carrier board specification](https://developer.nvidia.com/downloads/assets/embedded/secure/jetson/orin_nano/docs/jetson_orin_nano_devkit_carrier_board_specification_sp.pdf)
