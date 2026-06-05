# LOAM2_Boards_Winter/Spring_2026

## Description
These files contain the progress done on drone hardware for the LOAM 2 project. The current strategy is to produce a support board that connects to the Pixhawk's FMU, communicates with the nodes, and hosts many other vital systems. This board should concentrate as many of the drone peripherals as possible in a single PCB to ensure robustness. The support board contains the following sub-systems:

 * Power management and battery monitoring (PM)
 * GPS
 * ARM Switch
 * nRF54L15 (MCU)
 * nRF21540 (FEM)
 * GPS patch antenna
 * Bluetooth antenna (not certain yet)
 * JST Connectors for the FMU harness
 * Debug taps for all outputs

## Power Management

The power management issue is mostly solved. The board stored in `power_management_tests` was produced and thoroughly tested. We confirmed its outputs were the expected 3.3 and 5V, it handled the throuttle of the drone motors, and voltage and current readings are in agreement with lab equipment. **Note:** To ensure proper voltage and current readings, the resistor ratios implemented in the voltage and current sensors must be taken into account by the software. In Ardupilot, simply set `BATT_VOLT_MULT = 5.667` and `BATT_AMP_PERVLT = 25.0`. The chosen buck-converters can output currents of up to 3 A, far more than the FMU or the support board systems might draw. 

When redoing the layout for this board in the support board, make sure to follow IC datasheet recommendations and lay thick traces in the battery input and output ports, which can draw large currents (~80 A). 

## GPS

The GPS schematics were adapted from this repository: https://github.com/ZIOCC/Zio-Qwiic-GPS-Module-U-blox-NEO-M8N-0-10). They leverage the NEO-M8N-0 chip and a patch antenna at the GPS band of 1575.42 MHz. 

## Arm Switch

This replaces the floating ARM switch in the Pixhawk. 

## MCU and FEM

The drone communicates with node sensors through the nRF54L15 MCU using BLE. Currently, we assume the nodes will also host this MCU, as inter-communication between this IC is simple. The MCU is powered by the 3.3 V power line produced by the PM. Communication with the FMU is done through the TELEM2 port, which connects through UART to pins on the suppoart board MCU. 

The current drone wake-up strategy is to rely on energy harvesting to charge the base of an $npn$ transistor (2N3904) above its $V_{th}$, flipping the collector voltage from 3.3 V to 0 when charged, thus waking up the node. The node MCU must handle this and a few nuances regarding oscillation (described in our report) in firmware. We used the AEM30940 DK for our energy harvesting tests on the chosen band of 2.45 GHz. To ensure enough energy can be transmitted to charge the base of the transistor to around 0.65 V, a decent amount of power must be received. The The 47 C.F.R. § 15.247 permits a maximum of 1 W (30 dBm) of transmission power for the 2.45 GHz band given a few constraints, and the nRF54L15 can ony output a maximum of 8 dBm. To increase transmission power, we connect an RF FEM, the nRF21540, a bidirectional FEM (LNA for Rx and PA for Tx, with shiwtching controlled by GPIO pins or SPI). The antenna choice is still a major question for this transmission. 

The drone MCU in the suppoart board is programmed through an SWD interface. 

## Connectors and Debug Taps

The support board hosts multiple connectors that should be tightly packed in a harness connected to the FMU. These include the following ports: GPS, TELEM2, SWITCH, and POWER. Our initial connector selection was a DF13, but these are not fully compatible. We believe the correct connectors are JST. For testing and debugging reasons, it is helpful to pack debup taps for many essential nodes in the system (e.g. 3V3, 5V, CURRENT_SENSE, RX_GPS, etc). 


