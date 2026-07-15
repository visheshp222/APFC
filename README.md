#  Automatic Power Factor Correction (APFC) System

A MATLAB/Simulink-based **Automatic Power Factor Correction (APFC)** system that continuously monitors the power factor of a three-phase inductive load and automatically switches capacitor banks to improve the power factor toward a predefined target.

---

##  Overview

Industrial loads such as **induction motors, transformers, and other inductive equipment** consume reactive power from the electrical grid. This results in a lagging power factor, increased current, higher transmission losses, and reduced system efficiency.

This project implements an automatic power factor correction system that:

* Measures three-phase voltage and current
* Calculates the real-time power factor
* Detects low or over-corrected power factor
* Automatically switches capacitor banks
* Maintains the power factor close to a desired target

The system uses a **closed-loop control strategy** to dynamically compensate for the reactive power consumed by the inductive load.

---

##  Project Objective

The objective of this project is to automatically improve the power factor of a three-phase inductive load.

The system operates according to the following principle:

```text
Low Power Factor
        ↓
Measure Voltage and Current
        ↓
Calculate Power Factor
        ↓
Compare with Target Power Factor
        ↓
Switch Capacitor Banks
        ↓
Compensate Reactive Power
        ↓
Improve Power Factor
```

The controller continuously repeats this process to maintain the power factor within the desired operating range.

---

##  System Architecture

```text
                         ┌──────────────────────┐
                         │    3-PHASE SOURCE    │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │      SOURCE BUS      │
                         └──────────┬───────────┘
                                    │
                                    ▼
                    ┌──────────────────────────────┐
                    │  THREE-PHASE V-I MEASUREMENT │
                    └──────────────┬───────────────┘
                                   │
                    ┌──────────────┴──────────────┐
                    │                             │
                    ▼                             ▼
             ┌─────────────┐              ┌───────────────┐
             │  INDUCTIVE  │              │ CAPACITOR BANK │
             │    LOAD     │              │               │
             │             │              │   Stage 1     │
             │   Motors    │              │   Stage 2     │
             │ Transformers│              │   Stage 3     │
             └─────────────┘              └───────┬───────┘
                                                  │
                                                  ▼
                                          ┌───────────────┐
                                          │    BREAKERS    │
                                          └───────┬───────┘
                                                  │
                                                  ▼
                                          ┌───────────────┐
                                          │ APFC CONTROLLER│
                                          └───────┬───────┘
                                                  │
                                                  ▼
                                          ┌───────────────┐
                                          │    PF METER    │
                                          └───────────────┘
```

The APFC system works as a feedback control system:

```text
Measure → Calculate PF → Compare → Switch Capacitors → Correct PF
```

---

##  Measurement System

The Three-Phase V-I Measurement block is placed directly in the power circuit.

### Electrical Connections

```text
Three-Phase Source
        ↓
   A, B, C terminals
        ↓
 V-I Measurement Block
        ↓
   a, b, c terminals
        ↓
   Inductive Load
```

The measurement block provides two important signal outputs:

```text
Vabc → Three-phase voltage signals
Iabc → Three-phase current signals
```

These signals are supplied to the power factor measurement system.

The measurement block should be configured to output:

```text
Voltages and currents
```

---

##  Power Factor Calculation

The true power factor is calculated as:

```text
Power Factor = Real Power / Apparent Power
```

or:

```text
PF = P / S
```

where:

```text
P = Real Power
S = Apparent Power
```

### Real Power

The instantaneous three-phase power is:

```text
p(t) = Va × Ia + Vb × Ib + Vc × Ic
```

The average value of this instantaneous power over one complete AC cycle gives the real power:

```text
P = Average of p(t) over one complete cycle
```

### RMS Voltage and Current

The RMS values are calculated independently for each phase:

```text
Vrms = sqrt(average of V²)
```

```text
Irms = sqrt(average of I²)
```

The three-phase apparent power is then calculated as:

```text
S = Va_rms × Ia_rms
  + Vb_rms × Ib_rms
  + Vc_rms × Ic_rms
```

Therefore, the final power factor is:

```text
PF = P / S
```

This approach calculates the **true power factor** using measured real power and apparent power.

---

##  APFC Control Strategy

The APFC controller uses a **hysteresis-based control strategy**.

Hysteresis prevents continuous switching of capacitor banks when the power factor is close to the target value.

The control logic is:

```text
IF PF < Target PF - Hysteresis
        ↓
Switch ON one capacitor stage
```

```text
IF PF > Target PF + Hysteresis
        ↓
Switch OFF one capacitor stage
```

```text
IF Target PF - Hysteresis
   ≤ PF ≤
   Target PF + Hysteresis
        ↓
Maintain the current capacitor state
```

For example:

```text
Target PF  = 0.98
Hysteresis = 0.01
```

The controller operates as follows:

```text
PF < 0.97
    → Switch ON capacitor stage

0.97 ≤ PF ≤ 0.99
    → Maintain current state

PF > 0.99
    → Switch OFF capacitor stage
```

This control method prevents unnecessary capacitor switching and reduces relay or breaker chatter.

---

##  Capacitor Sizing

The required capacitor reactive power can be calculated using:

```text
Qc = P × [tan(cos⁻¹(PF₁)) - tan(cos⁻¹(PF₂))]
```

where:

```text
Qc  = Required capacitive reactive power
P   = Active power of the load
PF₁ = Initial power factor
PF₂ = Desired power factor
```

The capacitor bank supplies leading reactive power to compensate for the lagging reactive power consumed by the inductive load.

The total required compensation can be divided into multiple capacitor stages.

The capacitor stages should be selected carefully to prevent over-correction.

---

##  Controller Timing

The controller operates once per electrical cycle.

The duration of one AC cycle is:

```text
T = 1 / f
```

For a 50 Hz system:

```text
T = 1 / 50
T = 0.02 seconds
T = 20 ms
```

Updating the controller once per cycle provides a stable measurement and helps prevent unnecessary switching.

---

##  Expected System Behavior

The APFC system should follow this general sequence:

```text
Initial State
    ↓
Low Power Factor
    ↓
Controller Detects Lagging PF
    ↓
Capacitor Stage 1 Turns ON
    ↓
Power Factor Improves
    ↓
Additional Compensation Required?
    ↓
Capacitor Stage 2 Turns ON
    ↓
Power Factor Improves Further
    ↓
Target PF Reached
    ↓
System Maintains Current State
```

If the system becomes over-corrected:

```text
Leading / Excessive PF
        ↓
Controller Detects Over-Correction
        ↓
One Capacitor Stage Turns OFF
        ↓
Power Factor Returns Toward Target
```

The final power factor depends on:

* Load characteristics
* Capacitor bank ratings
* Hysteresis value
* Switching delay
* Measurement accuracy
* Simulation time step

---

##  Important Design Considerations

### Capacitor Overshoot

If the capacitor stages are too large, the system may overcompensate the inductive load.

This can cause the power factor to become leading.

Therefore, the capacitor bank sizes should be selected according to the expected reactive power demand.

---

### Hysteresis

Hysteresis is essential for stable APFC operation.

Without hysteresis, the controller may repeatedly switch capacitor stages ON and OFF when the power factor is close to the target value.

This behavior is known as:

```text
Chattering
```

A suitable hysteresis band provides stable operation.

---

### Measurement Window

The power factor measurement is averaged over one complete AC cycle.

For a 50 Hz system:

```text
One cycle = 20 ms
```

This provides a balance between measurement stability and controller response speed.

---

### Simulation Time Step

A sufficiently small simulation time step is required for accurate simulation of electrical switching behavior.

The appropriate time step depends on:

* Switching devices
* Circuit topology
* Simulation solver
* Required accuracy

---

##  Current Limitations

The current implementation uses:

* Discrete capacitor stages
* Breaker-based capacitor switching
* One-cycle measurement averaging
* Hysteresis-based control

Since the capacitor banks are switched in discrete steps, the final power factor may not reach exactly the target value.

---

##  Future Improvements

###  Thyristor-Switched Capacitors

Mechanical breakers can be replaced with **Thyristor-Switched Capacitors (TSCs)** for:

* Faster switching
* Reduced mechanical wear
* Improved dynamic response
* Reduced switching transients

---

###  Stateflow-Based Controller

The APFC controller can be implemented using Stateflow to create a visual state-machine-based control system.

This can make the control logic easier to understand and extend.

---

###  Performance Monitoring

Future versions can monitor and log:

```text
Power Factor
Active Power
Reactive Power
Apparent Power
Source Current
Capacitor Stage
```

These values can be used to generate performance graphs and analyze the effectiveness of the APFC system.

---

###  Intelligent Control

Advanced control methods can be implemented, including:

* Fuzzy Logic Control
* Adaptive Control
* Neural Network Control
* Model Predictive Control

These techniques can improve the response of the system under rapidly changing load conditions.

---

###  Automatic Capacitor Sizing

An advanced version of the system could automatically determine the required capacitor bank rating based on the measured load conditions.

---

##  Engineering Concepts Demonstrated

This project demonstrates the practical application of:

* Three-phase AC systems
* Reactive power compensation
* Power factor correction
* Capacitor banks
* Electrical measurements
* Feedback control systems
* Hysteresis control
* MATLAB/Simulink modeling
* Simscape Electrical
* Power system simulation

---


