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
    * [2.2 Generation of CTAT Voltage](#22-ctat-voltage-generation)
    * [2.3 Generation of PTAT Voltage](#23-ptat-voltage-generation)
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


---------------------------------------------------------------------------------------------------------------------------

## 1.Importance of BGR ##

In integrated circuits, many analog and mixed-signal blocks require a precise and stable reference voltage to operate correctly. However, voltages inside a chip usually vary with temperature, supply voltage, and process variations, which can affect circuit performance. A Bandgap Reference is used to overcome these variations and provide a stable reference voltage.

<img width="560" height="218" alt="BGR1" src="https://github.com/user-attachments/assets/3365215e-8e83-4ba0-9287-b28f019eb694" />

### Various Voltage Reference Techniques

| Feature | Resistive Divider | Zener Reference |
|--------|-------------------|-----------------|
| Temp. Stability | Poor (moves with R) | Moderate | 
| Supply Sensitivity | High (scales with VDD) | Moderate | 
| Operating Voltage | Any | High (>5 V) | 
| Power Efficiency | Constant drain | High current needed | 

Among all the techniques BGR provides constant and PVT independent reference voltage which is used for low power applications.
## Key Features of Bandgap Reference (BGR) #

### Features of Bandgap Reference (BGR)

| Feature | Description |
|--------|-------------|
| Temperature Independence | Produces a nearly constant reference voltage by combining CTAT (V_BE) and PTAT (ΔV_BE) voltages generated using Bipolar Junction Transistor (BJT) devices. |
| Stable Output Voltage | Produces a reference voltage close to the silicon bandgap (~1.2 V) which remains stable across temperature variations |
| Low Supply Sensitivity | Output voltage is minimally affected by changes in the supply voltage |
| High Accuracy | Uses ratio matching of devices and resistors, reducing the impact of process variations |

## Applications Of BGR ##

* Power Management Circuits such as LDO
* Data Converters such as ADC and DAC
* Biasing Circuits in Analog Systems
* Battery-Powered Devices

-------------------------------------------------------------------------------------------------------------------------

## 2. Bandgap Reference — Theory
-------------------------------------------------------------------------------------------------------------------------
  # 2.1 The Principle of BGR

  BGR is a combination of two voltages with opposite temperature coefficients that produces a temperature independent    reference voltage.
 * The base–emitter voltage of a Bipolar Junction Transistor (BJT) decreases with increasing temperature. This behavior is called Complementary To Absolute Temperature (CTAT) and typically has a slope of about −2 mV/°C.
 * The difference between the base–emitter voltages of two BJTs operating at different current densities, denoted as 
ΔVBE,increases with temperature. This voltage is Proportional To Absolute Temperature (PTAT) with a slope of 0.085mV/°C.

<img width="749" height="455" alt="BGR_Principle" src="https://github.com/user-attachments/assets/f0957b26-933d-4ba1-8275-742ad314c6f2" />Scaling the PTAT slope up by a constant K ≈ 26 and adding it to the CTAT gives a flat output:

$$
V_{REF} = V_{BE} + k \cdot \Delta V_{BE}
$$

The two opposing slopes cancel, yielding a residual tempco of 10–50 ppm/°C — ideal for a precision reference.

The output voltage of a Bandgap Reference is approximately 1.2 V because it is derived from the bandgap energy of silicon. The bandgap energy of silicon is about 1.205 eV at 0 K, which corresponds to a voltage of roughly 1.2 V when expressed in electrical terms.

----------------------------------------------------------------------------------------------------------------------------
## 2.2 Generation of CTAT Voltage ##
<img width="668" height="249" alt="CTAT" src="https://github.com/user-attachments/assets/b46d69d3-3646-4851-9f21-f81fbaee7017" />

The diode connected PNP transistor is forward biased by a constant current source Io, then voltage across VBE is given by

$$
V_BE = V_T \ln\left(\frac{I_0}{I_S}\right)
$$

Where  VT  is given by

$$
V_T = \frac{kT}{q}
$$

The saturation current is given by

$$
I_S = A T^{(4+m)} e^{-\frac{E_g}{kT}}
$$

$$
\mu \propto \mu_0 T^m
$$

m is a constant

$$
m = -\frac{3}{2}
$$

When differentiating with respect to temprature

$$
\frac{dV}{dT} =
\frac{V_D - (4+m)V_T - \frac{E_g}{q}}{T}
$$

Where T=300K, VBE is 0.7V

$$
\frac{dV}{dT} =
\frac{0.7 - (4 - 1.5)(0.026) - 1.2}{300}
$$

$$
\frac{dV}{dT} \approx -1.88 \; mV/^\circ C
$$

This negative slope represents the CTAT (Complementary To Absolute Temperature) behavior used in a Bandgap Reference.

-----------------------------------------------------------------------------------------------------------------------------

## 2.3 Generation of PTAT Voltage

<img width="804" height="405" alt="PTAT" src="https://github.com/user-attachments/assets/bbcfc00d-ba32-486f-8caf-3c3179dafb1e" />


<img width="641" height="366" alt="Screenshot 2026-03-11 120506" src="https://github.com/user-attachments/assets/73d3755f-e004-477e-a140-4e2180c4cc8d" />

---------------------------------------------------------------------------------------------------------------------------

## 2.4 Types of BGR

Architecture wise BGR can be designed in 2 ways
* Using Self-biased Current Mirror
* Using Operational-amplifier


Application wise BGR can be categorized as
* Low-Voltage BGR
* Low-Power BGR
* High-PSRR and low-noise BGR
* Curvature Compensated BGR

In this project we developed a Self-biased current mirror BGR with Tempco of less than 30ppm/°C
  


