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
**Magic**

Magic is a VLSI layout tool.

**Steps to install Magic** - Open the terminal and type the following to install Magic
```bash
$  wget http://opencircuitdesign.com/magic/archive/magic-8.3.32.tgz
$  tar xvfz magic-8.3.32.tgz
$  cd magic-8.3.28
$  ./configure
$  sudo make
$  sudo make install
```

**Netgen**
Netgen is a tool for comparing netlists, a process known as LVS, which stands for "Layout vs. Schematic". This is an important step in the integrated circuit design flow, ensuring that the geometry that has been laid out matches the expected circuit.
**Steps to install Netgen** - Open the terminal and type the following to insatll Netgen.
```bash
$  git clone git://opencircuitdesign.com/netgen
$  cd netgen
$  ./configure
$  sudo make
$  sudo make install 
```

--------------------------------------------------------------------------------------------------------------------------

## 5.2 PDK Setup ##

A process design kit (PDK) is a set of files used within the semiconductor industry to model a fabrication process for the design tools used to design an integrated circuit. The PDK is created by the foundry defining a certain technology variation for their processes. It is then passed to their customers to use in the design process.

The PDK we are going to use for this BGR is Google Skywater-130 (130 nm) PDK.

```bash
$  git clone git://opencircuitdesign.com/netgen
$  cd netgen
$  ./configure
$  sudo make
$  sudo make install 
``` 
**Design Flow Overview:**

```bash
Schematic Design   ──[Ngspice + Sky130 models]──►  Pre-Layout Simulation
       │
       ▼
 Layout Design     ──[Magic + Sky130 tech file]──►  DRC + PEX
       │
       ▼
      LVS          ──[Netgen + Sky130 rule file]──►  Schematic vs Layout
       │
       ▼
Post-Layout Sim    ──[Ngspice + extracted netlist]──►  Final Verification
``` 
----------------------------------------------------------------------------------------------------------------------

## 6. Writing a SPICE Netlist ##
---------------------------------------------------------------------------------------------------------------------

A SPICE netlist is a text file that represents an electronic circuit by listing its components, node connections, and simulation instructions in a format that can be understood by Ngspice. Each .sp file in this project is organized using a consistent structure so that the simulator can correctly interpret and run the circuit simulations.

-----------------------------------------------------------------------------------------------------------------------

## 6.1 Netlist Structure ##
```bash
*  Title / Comment line (must be the first line)

*  ----------------------------
*  1. Include PDK model files
*  ----------------------------
.lib "/path/to/sky130.lib.spice" tt

*  ----------------------------
*  2. Global supply nodes
*  ----------------------------
.global gnd vdd
vdd vdd gnd 1.8

*  ----------------------------
*  3. Circuit components
*  ----------------------------
* Syntax: ComponentName  Node+ Node-  ModelName  Parameters

* BJT (PNP)
xqp1  vdd  net1  net1  sky130_fd_pr__pnp_05v5_W3p40L3p40  m=1

* MOSFET (PFET)
xmp1  net1  net2  vdd  vdd  sky130_fd_pr__pfet_01v8_lvt  l=2 w=5 m=4

* MOSFET (NFET)
xmn1  net2  net3  gnd  gnd  sky130_fd_pr__nfet_01v8_lvt  l=1 w=5 m=8

* Resistor
xra1  net3  net4  sky130_fd_pr__res_high_po_1p41  l=7.8 w=1.41 m=1

*  ----------------------------
*  4. Simulation commands
*  ----------------------------
.dc temp -40 125 5        * DC sweep: temperature from -40 to 125°C in steps of 5

.control
  run
  plot v(vref)            * Plot the reference voltage node
.endc

.end                      * Every netlist must end with .end
```

-----------------------------------------------------------------------------------------------------------------------------
## 6.2 Key Rules ##

* First line is always a comment
* Component names are case-insensitive
* X prefix = subcircuit call
* Node 0 or gnd = ground
* .end is mandatory

-----------------------------------------------------------------------------------------------------------------------------

## 6.3 Sky130 Device Instantiation ##

Sky130 devices are defined as subcircuits in the PDK model files, so they are called using X (subcircuit instance) syntax:

**PNP BJT**
```bash
 xInstanceName  Collector  Base  Emitter  ModelName  m=<multiplier>
xqp1  gnd  net1  net1  sky130_fd_pr__pnp_05v5_W3p40L3p40  m=1
```
**PFET**
```bash
 xInstanceName  Drain  Gate  Source  Bulk  ModelName  l=<> w=<> m=<>
xmp1  net1  net2  vdd  vdd  sky130_fd_pr__pfet_01v8_lvt  l=2 w=5 m=4
```
**NFET**
```bash
 xInstanceName  Drain  Gate  Source  Bulk  ModelName  l=<> w=<> m=<>
xmn1  net2  net3  gnd  gnd  sky130_fd_pr__nfet_01v8_lvt  l=1 w=5 m=8
```
**RPOLYH Resistor**
```bash
 xInstanceName  Node+  Node-  ModelName  l=<> w=<> m=<>
xra1  net3  net4  sky130_fd_pr__res_high_po_1p41  l=7.8 w=1.41 m=1
```
---------------------------------------------------------------------------------------------------------------------------

## 6.4 Simulation Commands ##
* .dc temp -40 125 1 used to sweep temperature with step of 1
* .dc vdd 1 3.3 0.1  used to sweep voltage with step of 0.1
* .control / .endc used to  Interactive block for run and plot commands
* .lib to include PDK files for specific corner (tt,ff,ss)

-------------------------------------------------------------------------------------------------------------------------

## 6.5 Example Netlists ##

**BGR with SBCM (bgr_lvt_rpolyh_3p40.sp)**
 ```bash
* BGR — Self-Biased Current Mirror (TT corner)

.lib "/home/vsduser/Desktop/Work/eda-technology/sky130/libs/sky130.lib.spice" tt

.global gnd vdd
vdd vdd gnd 1.8

* ---- PMOS Current Mirror ----
xmp1  net1  net2  vdd  vdd  sky130_fd_pr__pfet_01v8_lvt  l=2 w=5 m=4
xmp2  net2  net2  vdd  vdd  sky130_fd_pr__pfet_01v8_lvt  l=2 w=5 m=4
xmp3  vref  net2  vdd  vdd  sky130_fd_pr__pfet_01v8_lvt  l=2 w=5 m=4

* ---- NMOS (SBCM) ----
xmn1  net1  net1  gnd  gnd  sky130_fd_pr__nfet_01v8_lvt  l=1 w=5 m=8
xmn2  net2  net1  net3  gnd  sky130_fd_pr__nfet_01v8_lvt  l=1 w=5 m=8

* ---- CTAT Branch (Q1 x1) ----
xqp1  gnd  net1  net1  sky130_fd_pr__pnp_05v5_W3p40L3p40  m=1

* ---- PTAT Branch (R1 + Q2 x8) ----
xra1  net3  net4  sky130_fd_pr__res_high_po_1p41  l=7.8 w=1.41 m=2
xqp2  gnd  net4  net4  sky130_fd_pr__pnp_05v5_W3p40L3p40  m=8

* ---- Reference Branch (R2 + Q3 x1) ----
xrb1  vref  net5  sky130_fd_pr__res_high_po_1p41  l=7.8 w=1.41 m=16
xqp3  gnd  net5  net5  sky130_fd_pr__pnp_05v5_W3p40L3p40  m=1

* ---- Start-up Circuit ----
xmp4  net6  net6  vdd  vdd  sky130_fd_pr__pfet_01v8_lvt  l=2 w=5 m=1
xmp5  net7  net6  vdd  vdd  sky130_fd_pr__pfet_01v8_lvt  l=2 w=5 m=1
xmn3  net7  net7  gnd  gnd  sky130_fd_pr__nfet_01v8_lvt  l=7 w=1 m=1
xmn4  net6  net7  gnd  gnd  sky130_fd_pr__nfet_01v8_lvt  l=7 w=1 m=1

.dc temp -40 125 5

.control
  run
  plot v(vref)
.endc

.end
```

**Changing Corner**
To run FF or SS corners, change only the .lib tag — everything else stays the same:

```bash
.lib "/path/to/sky130.lib.spice" ff   * Fast-Fast
.lib "/path/to/sky130.lib.spice" ss   * Slow-Slow
```
-----------------------------------------------------------------------------------------------------------------------------

## 7. Pre-Layout Simulation ##
---------------------------------------------------------------------------------------------------------------------------
All spice files are simulated using ngspice

## 7.1 CTAT Simulations

**Single BJT CTAT**
```bash
ngspice ctat_voltage_gen.sp
```
The base-emitter voltage of a single PNP BJT biased at a constant current decreases linearly with temperature at approximately −2 mV/°C — classic CTAT behaviour.


<img width="739" height="495" alt="ctat_voltage_gen" src="https://github.com/user-attachments/assets/c0f819f8-be83-4cd4-be37-87bf8946ef6d" />
<img width="708" height="550" alt="ctat_voltage_gen_" src="https://github.com/user-attachments/assets/7165e093-5bbd-4531-9d47-853c12888ec3" />

---------------------------------------------------------------------------------------------------------------------------

**Multiple BJT CTAT**
```bash
ngspice ctat_voltage_gen_mul_bjt.sp
```
With 8 BJTs in parallel in branch 2 vs 1 BJT in branch 1, the CTAT slope of the branch 1 node becomes steeper (more negative), as expected from the BJT area ratio.

<img width="611" height="549" alt="ctat_voltage_gen_mul_bjts_2" src="https://github.com/user-attachments/assets/dacb6f24-701e-4a1d-a0dc-eff7b68f3bb7" />
<img width="736" height="483" alt="ctat_voltage_gen_mul_bjts" src="https://github.com/user-attachments/assets/9ea504c7-6b26-45b6-a9c2-cc7995718dd6" />

--------------------------------------------------------------------------------------------------------------------------
**CTAT with Varying Current**

```bash
ngspice ctat_voltage_gen_var_current.sp
```
<img width="706" height="547" alt="ctat_vol_gen_var_currents_1" src="https://github.com/user-attachments/assets/dd6de928-1b05-4a68-a9cc-fe7355b51688" />
<img width="740" height="493" alt="ctat_vol_gen_var_currents" src="https://github.com/user-attachments/assets/b8242cca-349e-4bc3-80ee-48eba47d78cf" />

Reducing bias current lowers V_BE and increases the magnitude of the negative slope — 
confirming the I₀ dependence in the CTAT equation.

--------------------------------------------------------------------------------------------------------------------------
## 7.2 PTAT Simulations ##
-----------------------------------------------------------------------------------------------------------------------------
**PTAT with Ideal Voltage Source (VCVS)**
```bash
ngspice ptat_voltage_gen.sp
```
<img width="740" height="489" alt="ptat_vol_gen_ckt" src="https://github.com/user-attachments/assets/41a942ae-4369-4035-974f-dbd398829ece" /><img width="707" height="559" alt="ptat_vol_gen_ckt_3" src="https://github.com/user-attachments/assets/acd6d427-c070-42a1-89a1-1172c6ccca6e" />
<img width="1434" height="550" alt="ptat_vol_gen_ckt_2" src="https://github.com/user-attachments/assets/a4b777a4-c423-4f40-b84c-534c7fdfc5ec" />


Using a VCVS to force equal voltages at the branch nodes, the voltage across R1 is proportional to V_T ln(N) — confirming the PTAT slope of +0.085 mV/°C per unit of ln(N).

--------------------------------------------------------------------------------------------------------------------------

## PTAT with Ideal Current Source ##

```bash
ngspice ptat_voltage_gen_ideal_current_source.sp
```
<img width="1434" height="550" alt="ptat_vol_gen_ckt_2" src="https://github.com/user-attachments/assets/1e7e92cf-d873-40ec-8f98-1d30061ec9a2" />

--------------------------------------------------------------------------------------------------------------------------
## 7.3 Resistor Tempco ##

```bash
ngspice res_tempco.sp
```
<img width="1229" height="514" alt="Screenshot 2026-03-11 134247" src="https://github.com/user-attachments/assets/ff66e6cd-5b94-4974-a92b-4728d57a3741" />
<img width="736" height="476" alt="Res_tempco_var_cur" src="https://github.com/user-attachments/assets/969e86df-8dbb-445d-8fd4-e0039c11d10b" />

-----------------------------------------------------------------------------------------------------------------------

## Resistor Tempco with Varying Current ##

```bash
ngspice res_tempco_var_current.sp
```
<img width="1403" height="548" alt="Res_tempco_var_current_1" src="https://github.com/user-attachments/assets/6e4db788-0c62-436b-ac06-a693629b77a4" />
<img width="739" height="488" alt="Res_tempco_var_current" src="https://github.com/user-attachments/assets/3e4bc89f-0458-4027-bcae-9af88b4bf118" />

-----------------------------------------------------------------------------------------------------------------------------
## 7.4 FET Tempco ##


```bash
ngspice fet_tempco.sp
```
<img width="1445" height="548" alt="FET_tempco" src="https://github.com/user-attachments/assets/457ba527-cfce-4781-9046-c14c8bbfd587" />

-------------------------------------------------------------------------------------------------------------------------

## 7.5 Full BGR Simulations ##

**BGR with Ideal Op-Amp**

```bash
ngspice bgr_using_ideal_opamp.sp
```

Before committing to the self-biased current mirror, the full BGR loop is verified using a VCVS as an ideal op-amp. The output should be an umbrella-shaped curve centred near 1.2 V — confirming that the CTAT + PTAT cancellation is working correctly.

<img width="1854" height="984" alt="bgr_using_ideal_opamp_var_supply1" src="https://github.com/user-attachments/assets/cb338c5d-a870-49bc-90f5-baa82355a692" />
<img width="1420" height="556" alt="bgr_using_ideal_opamp_var_supply" src="https://github.com/user-attachments/assets/ec3e2a48-87d6-4d0a-b2f2-6b194161b826" />
<img width="1412" height="553" alt="bgr_using_ideal_opamp_2" src="https://github.com/user-attachments/assets/fc1695ad-2865-4127-8619-30c34f624afb" />
<img width="745" height="499" alt="bgr_using_ideal_opamp" src="https://github.com/user-attachments/assets/8d4b576c-fcbe-4b36-96da-bbc426e0382f" />

From graph

Vmax=1.23583V       Vmin=1.23175V  Vnominal=1.23568V  Tmax=125   Tmin=-40

Hence Tempco of BGR using ideal opamp is 20 ppm/c

-----------------------------------------------------------------------------------------------------------------------------
## BGR with SBCM — Typical (TT corner) ##

```bash
ngspice bgr_lvt_rpolyh_3p40.sp
```

The self-biased current mirror replaces the ideal op-amp. Expected result:
* V_REF ≈ 1.2 V, flat across −40 °C to +125 °C
* Tempco ≈ 21.7 ppm/°C

<img width="1413" height="550" alt="bgr_lvt_rpolyh_3p40_1" src="https://github.com/user-attachments/assets/a85f8d63-4f2a-4e27-9036-0b169ab29767" />
<img width="1450" height="550" alt="bgr_lvt_rpolyh_3p40" src="https://github.com/user-attachments/assets/b0e6694a-cfa1-47b2-8e3c-bba2dd80a34f" />

Vmax=1.10999,   Vmin=1.10569,  Vnom=1.10994, Tmax=125, Tmin=-40

Tempco=23 ppm/c

**Transient simulations**
<img width="1855" height="982" alt="bgr_lvt_rpolyh_3p40 _trans" src="https://github.com/user-attachments/assets/a3af5eb9-8f7b-4970-afb0-4541607b038e" />

We are giving the pulse of 0 to 2V with delay of 10ns, rise time of 1us, fall time of 1us, period of 1ms, and  pulse width of 100us.

Transient analysis is done with a step size of 5n, and till 10u 

Vdd is varying from 0 to 2V with in 1uS
Blue line Vref and is constant at 1.1V. Initially Vref is following Vdd and after some time it is stable around @1.1us.
Green one is net2 voltage. Initially it is followed by Vdd and after some time net2 voltage is down.
Other voltages also stable around at 1.1us.

So we can say that start up time is 1.1us

From this graph we can easily found how startup circuit is work.

--------------------------------------------------------------------------------------------------------------------------

## BGR — Fast-Fast (FF) Corner ##

```bash
ngspice bgr_lvt_rpolyh_3p40_ff.sp
```
Tempco ≈ 10 ppm/°C (best case — process corners tend to improve tempco here)<img width="1406" height="550" alt="bgr_lvt_rpolyh_3p40_ff_2" src="https://github.com/user-attachments/assets/71d0e379-2051-459d-8cad-10868544e979" />
<img width="1401" height="551" alt="bgr_lvt_rpolyh_3p40_ff_1" src="https://github.com/user-attachments/assets/98ae0398-3ef8-431e-ad59-3718926a0448" />


Vmax=1.12227, Vmin=1.12038, Vnom=1.12205, Tmax-Tmin=165
TC=10ppm/c

This is the best corner because it is internally compensated due to that  resistance tempco and current tempco combined.



<img width="1451" height="574" alt="bgr_lvt_rpolyh_3p40_ff" src="https://github.com/user-attachments/assets/9914b59e-0ba2-435b-8d4a-1bdf48b73358" />

**Transient simulations**
<img width="1448" height="564" alt="bgr_lvt_rpolyh_3p40_ff_trans" src="https://github.com/user-attachments/assets/e94b6d75-ebd7-48dd-9913-bab08eff3567" />

--------------------------------------------------------------------------------------------------------------------------

## BGR — Slow-Slow (SS) Corner ##

```bash
ngspice bgr_lvt_rpolyh_3p40_ss.sp
```

Tempco ≈ 45 ppm/°C (worst case — still within the 50 ppm/°C specification)


<img width="1410" height="542" alt="bgr_lvt_rpolyh_3p40_ss_1" src="https://github.com/user-attachments/assets/c8fac7e2-dc09-4cf6-9d4d-84370440caa5" />
<img width="1448" height="542" alt="bgr_lvt_rpolyh_3p40_ss" src="https://github.com/user-attachments/assets/1715a817-0ea7-4c73-9184-3e785418063f" />


Vmax=1.0979V, Vmin=1.08936V, Vnom=1.09742
TC=47ppm/c

**Transient simulations**
<img width="1602" height="555" alt="bgr_lvt_rpolyh_3p40_ss_trans" src="https://github.com/user-attachments/assets/d4eb3d9d-88fd-4c94-9166-04b431de83ab" />

-------------------------------------------------------------------------------------------------------------------------

## BGR — Supply Variation ##

```bash
ngspice bgr_lvt_rpolyh_3p40_var_supply.sp
```
VDD swept from 1.62 V to 1.98 V (±10%). V_REF should remain stable, validating the PSRR of the SBCM topology.


<img width="703" height="550" alt="bgr_lvt_rpolyh_3p40_var_supply_2" src="https://github.com/user-attachments/assets/526afe15-5910-4c6b-9149-837ed361f8be" />
<img width="1411" height="550" alt="bgr_lvt_rpolyh_3p40_var_supply_1" src="https://github.com/user-attachments/assets/270b8053-9255-418b-b0fc-d4cb9cea707d" />
<img width="742" height="490" alt="bgr_lvt_rpolyh_3p40_var_supply" src="https://github.com/user-attachments/assets/a0506bd1-29bc-4014-96fe-cd286c5484ef" />

If we compare the results of Vref vs Variable power supply (VDD) with SBCM BGR and Ideal opamp based BGR then the sensitivity of Vref in opamp is more than the sensitivity of SBCM BGR.

**Transient simulations**
<img width="1438" height="562" alt="bgr_lvt_rpolyh_3p40_var_supply_trans" src="https://github.com/user-attachments/assets/c6fd3af2-b44b-45aa-8acf-a50c320d9644" />

<img width="1417" height="554" alt="bgr_lvt_rpolyh_3p40_var_supply_trans_4" src="https://github.com/user-attachments/assets/cb129d9c-d803-466e-b599-a694103f142d" />
<img width="1438" height="562" alt="bgr_lvt_rpolyh_3p40_var_supply_trans_2" src="https://github.com/user-attachments/assets/88e95569-9190-481a-80aa-870398c9876c" />
<img width="1421" height="562" alt="bgr_lvt_rpolyh_3p40_var_supply_trans_1" src="https://github.com/user-attachments/assets/0059f96d-c036-41ee-9c8c-719d1bbb9d54" />

It took around 1uS to start the simulation. It meets our specifications where start-up-time < 2uS.
Net2[PMOS] should be down and net1 [NMOS} should be up to start the current in BGR after start-up.
So blue line is net2 which is down and yellow one is net1 which is up after startup.

----------------------------------------------------------------------------------------------------------------------------

## Isolated start-up BGR ##

```bash
ngspice bgr_lvt_rpolyh_3p40.sp
```


During transient simulations if start up circuit is not  there then BGR is always at Zero-operating condition<img width="1849" height="984" alt="bgr_lvt_rpolyh_3p40_var_supply_isolated_startup_2" src="https://github.com/user-attachments/assets/6c5ec4f4-7241-4126-b3b6-4fb42d9f4425" />
<img width="1849" height="984" alt="bgr_lvt_rpolyh_3p40_isolated_startup_vref" src="https://github.com/user-attachments/assets/f98a892a-60b9-4eb0-bec8-2250cbe2a2ff" />
<img width="1849" height="984" alt="bgr_lvt_rpolyh_3p40_isolated_startup_current" src="https://github.com/user-attachments/assets/5154a265-1e1d-4fc9-ae67-cb5c3a677a63" />
<img width="1447" height="550" alt="bgr_lvt_rpolyh_3p40_isolated_startup" src="https://github.com/user-attachments/assets/7df199fe-dec7-4290-a192-5cecf283a44f" />

--------------------------------------------------------------------------------------------------------------------

## Cuurent through Start-Up circuit ##

```bash
ngspice bgr_lvt_rpolyh_3p40.sp
```

<img width="1439" height="564" alt="start-up_current_mp5" src="https://github.com/user-attachments/assets/5d09bf6b-72a8-4ce3-943d-65e1883d0cd7" />
<img width="1849" height="984" alt="bgr_lvt_rpolyh_3p40_startupckt_cur" src="https://github.com/user-attachments/assets/fad608fd-57f9-4941-9ebe-f828267b6ef7" />


----------------------------------------------------------------------------------------------------------------------------

