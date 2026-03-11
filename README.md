# Voltage Mode Bandgap Reference IP Design
**Sky130 PDK | ngspice | Magic | Netgen**

This project presents the design and simulation of a Bandgap Reference (BGR) circuit using open-source EDA tools. The BGR generates a temperature-independent reference voltage. The design is implemented and verified using ngspice, magic, and netgen tools.

-----------------------------------------------------------------------------------------------------------------------------
**Objective**
To design and verify a Bandgap Reference IP using open-source EDA tools that produces a stable temperature-independent reference voltage, validated through pre-layout and post-layout simulations for integration in analog circuits such as LDOs and mixed-signal systems using SkyWater 130nm PDK.

-------------------------------------------------------------------------------------------------------------------------------
**Introduction**
A Bandgap Reference (BGR) is an essential analog circuit used to generate a stable reference voltage that is largely independent of temperature, supply voltage, and process variations. It works by combining a negative temperature coefficient voltage (V_BE) with a positive temperature coefficient voltage (ΔV_BE) obtained from Bipolar Junction Transistor (BJT) devices. The weighted sum of these voltages produces a nearly constant voltage close to the silicon bandgap (~1.2 V).

-----------------------------------------------------------------------------------------------------------------------------

## Table of Contents

1. [Importance of BGR?](#1-why-a-bandgap-reference)
2. [Bandgap Reference — Theory](#2-bandgap-reference--theory)
    * [2.1 The Principle of BGR ](#21-the-bgr-principle)
    * [2.2 CTAT Voltage Generation](#22-ctat-voltage-generation)
    * [2.3 PTAT Voltage Generation](#23-ptat-voltage-generation)
    * [2.4 Types of BGR](#24-bgr-types)
3. [Self-Biased Current Mirror Based BGR](#3-self-biased-current-mirror-based-bgr)
    * [3.1 Self-Biased Current Mirror](#31-self-biased-current-mirror)
    * [3.2 Reference Branch Circuit](#32-reference-branch-circuit)
    * [3.3 Start-up Circuit](#33-start-up-circuit)
    * [3.4 Complete BGR Circuit](#34-complete-bgr-circuit)
4. [Design Specification and Device Datasheet](#4-design-specification-and-device-datasheet)
    * [4.1 Design Specifications](#41-design-specifications)
    * [4.2 Device Datasheet](#42-device-datasheet)
    * [4.3 Circuit Sizing](#43-circuit-sizing)
5. [Tools and PDK Setup](#5-tools-and-pdk-setup)
    * [5.1 Tool Setup](#51-tool-setup)
    * [5.2 PDK Setup](#52-pdk-setup)
6. [Writing a SPICE Netlist](#6-writing-a-spice-netlist)
    * [6.1 Netlist Structure](#61-netlist-structure)
    * [6.2 Key Rules](#62-key-rules)
    * [6.3 Sky130 Device Instantiation](#63-sky130-device-instantiation)
    * [6.4 Simulation Commands](#64-simulation-commands)
    * [6.5 Example Netlists](#65-example-netlists)
7. [Pre-Layout Simulation](#7-pre-layout-simulation)
    * [7.1 CTAT Simulations](#71-ctat-simulations)
    * [7.2 PTAT Simulations](#72-ptat-simulations)
    * [7.3 Resistor Tempco](#73-resistor-tempco)
    * [7.4 FET Tempco](#74-fet-tempco)
    * [7.5 Full BGR Simulations](#75-full-bgr-simulations)
8. [Layout Design](#8-layout-design)
    * [8.1 Leaf Cell Layouts](#81-leaf-cell-layouts)
    * [8.2 Block-Level Layouts](#82-block-level-layouts)
    * [8.3 Top-Level Layout](#83-top-level-layout)
9. [LVS Verification](#9-lvs-verification)
10. [Results Summary](#10-results-summary)
11. [References](#11-references)
