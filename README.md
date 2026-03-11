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

-----------------------------------------------------------------------------------------------------------------------------

## 3. Self-Biased Current Mirror Based BGR

BGR mainly contains of 5 sub circuits
* CTAT Voltage generation circuit
* PTAT Voltage generation circuit
* Self-biased current mirror circuit
* Reference branch circuit
* Start-up circuit

----------------------------------------------------------------------------------------------------------------------------

## 3.1 Self-Biased Current Mirror
<img width="1026" height="514" alt="Screenshot 2026-03-11 121908" src="https://github.com/user-attachments/assets/08048e32-f166-42f9-9148-0659da7b1fbf" />

---------------------------------------------------------------------------------------------------------------------------

## 3.2 Reference Branch Circuit
A third PMOS MP3 mirrors the same current I₃ ≈ I₁ ≈ I₂ into a reference branch consisting of R2 in series with a PNP BJT Q3.
The voltage across Q3 is CTAT, and the voltage across R2 (= R2 × I₃) is PTAT since I₃ is PTAT-proportional. Their sum is:


$$
V_{REF} = V_{Q3} + V_{R2}=V_{BE} + I.R2
$$

<img width="559" height="546" alt="Screenshot 2026-03-11 122919" src="https://github.com/user-attachments/assets/b2f749de-c889-4126-a969-952a576fde6c" />

-----------------------------------------------------------------------------------------------------------------------------
## 3.3 Start-up Circuit ##

The SBCM has two stable operating points:

* I_in = I_out = 0 A (degenerate / undesired)
* The desired bias current
  
At power-on, the circuit is at the zero-current point. The start-up circuit must:

* Detect the zero-current state and force the circuit out of it.
* Disengage once the correct operating point is reached — otherwise it will corrupt the bias.
  
Operation:

* Initially, all branch currents = 0 → net2 follows VDD.
* When net2 voltage exceeds net6 by one V_T, current flows through MP5 → net1 rises → MN1/MN2 turn on → circuit reaches the desired operating point.
* Once stable, the start-up circuit is self-defeating: net2 drops to the correct bias level, MP5 turns off, and the start-up path is isolated.

<img width="761" height="509" alt="Screenshot 2026-03-11 123303" src="https://github.com/user-attachments/assets/1a1da466-5542-41a2-9edd-26d44a2be01e" />

----------------------------------------------------------------------------------------------------------------------------

## 3.4 Complete BGR Circuit ##

<img width="751" height="490" alt="BGR" src="https://github.com/user-attachments/assets/c2327adb-5932-4528-bda0-92d885773a98" />

----------------------------------------------------------------------------------------------------------------------------

## 4. Design Specification and Device Datasheet
-----------------------------------------------------------------------------------------------------------------------------
## 4.1 Design Specifications

* VDD = 1.8V
* Power Consumption = < 60uW
* Operating Temperature = −40 °C to +125 °C
* Start-up circuit Current = < 2 µA
* Start-up Time = 2us
* TEMPCO of V_Ref = < 50 ppm/°C

----------------------------------------------------------------------------------------------------------------------------

## 4.2 Device Datasheet
  **PNP BJT**
  ### Device Parameters

| Parameter | NFET | PFET |
|-----------|------|------|
| Type | LVT | LVT |
| Voltage | 1.8 V | 1.8 V |
| Threshold Voltage | ~0.4 V | ~−0.6 V |
| Sky130 Model | sky130_fd_pr__nfet_01v8_lvt | sky130_fd_pr__pfet_01v8_lvt |

**MOSFET (LVT, 1.8V)**

| Parameter | NFET | PFET |
|-----------|------|------|
| Type | LVT | LVT |
| Voltage | 1.8 V | 1.8 V |
| Threshold Voltage | ~0.4 V | ~−0.6 V |
| Sky130 Model | sky130_fd_pr__nfet_01v8_lvt | sky130_fd_pr__pfet_01v8_lvt |


**Resistor (RPOLYH)**

| Parameter | Value |
|-----------|-------|
| Sheet Resistance | ~350 Ω/□ |
| Tempco | 2.5 Ω/°C |
| Available Bin Widths | 0.35 µm, 0.69 µm, 1.41 µm, 2.85 µm, 5.73 µm |
| Sky130 Model | sky130_fd_pr__res_high_po |

------------------------------------------------------------------------------------------------------------------------

## 4.3 Circuit Design ##

**1. Current Calculation**

Max power = 60 µW at VDD = 1.8 V → max total current = 33.33 µA.
With 3 branches: 10 µA/branch (3 × 10 = 30 µA, leaving headroom for start-up).

**2. BJT ratio N**
A moderate N = 8 BJTs in parallel in branch 2 gives

* Good layout matching (common-centroid array)
* Moderate R1 value (not too large, keeping area reasonable)

 **3. R1 calculation**

 <img width="1001" height="279" alt="Screenshot 2026-03-11 125013" src="https://github.com/user-attachments/assets/d4a8c912-c37a-4442-ac6f-589293d7fec6" />

 Implemented as: W = 1.41 µm, L = 7.8 µm, unit = 2 kΩ → 2 series + 2 parallel (2+2+(2‖2))

 **4. R2 calculation**
 <img width="1020" height="421" alt="Screenshot 2026-03-11 125117" src="https://github.com/user-attachments/assets/d5302bd0-74c0-4040-8040-a26541315d87" />

 Implemented as: 16 in series + 2 in parallel (2+2+...+2+(2‖2))

 **5 — PMOS sizing (SBCM)**

MP1, MP2: Both in saturation. Long channel (L = 2 µm) to reduce channel length modulation.
Final size: L = 2 µm, W = 5 µm, M = 4

**6 — NMOS sizing (SBCM)**

MN1, MN2: Operated in deep sub-threshold to achieve low current with compact area.
Final size: L = 1 µm, W = 5 µm, M = 8

 <img width="822" height="518" alt="finalbgr" src="https://github.com/user-attachments/assets/23c3b914-1407-4138-bfc3-df4c6ce1a70d" />


----------------------------------------------------------------------------------------------------------------------------

## 5. Tools and PDK Setup ##
-----------------------------------------------------------------------------------------------------------------------------
  **5.1 Tool Setup**

  For the design and simulation of the BGR circuit we will need the following tools.
  * Spice netlist simulation - Ngspice
  * Layout Design and DRC - Magic
  * LVS - Netgen
**Ngspice**

Ngspice is the open source spice simulator for electric and electronic circuits. Ngspice is an open project, there is no closed group of developers.

Ngspice Reference Manual: Complete reference manual in HTML format.

**Steps to install Ngspice** - Open the terminal and type the following to install Ngspice

```bash
sudo apt-get install ngspice
```
