# DC Microgrid Research — Modeling & Control

**MATLAB/Simulink | Simscape Electrical | Power Electronics | Control Systems**

**Status:** Ongoing  
**Current Stage:** PV + Battery integration completed  
**Next Stage:** Wind generation subsystem

## Overview

This project is a MATLAB/Simulink and Simscape Electrical DC microgrid modeling project focused on renewable energy integration, power electronics, local control, and system-level power sharing.

Developed as part of an independent research effort initiated under the guidance of a Concordia University power electronics professor, with the goal of studying DC microgrid modeling, renewable energy integration, and control.

The project began by first studying the fundamentals of microgrids and then building the electrical model progressively from simple DC circuits. Rather than immediately creating a complex system, each major component was modeled and tested separately before being integrated into a common DC microgrid.

The current model contains a photovoltaic (PV) subsystem and a battery energy storage subsystem connected to a common DC bus. The PV subsystem uses Perturb and Observe (P&O) Maximum Power Point Tracking (MPPT), while the battery subsystem uses closed-loop voltage control to support the DC bus near its 50 V reference.

The next planned major extension is the development and integration of a wind-generation subsystem.

---

## Project Objectives

The main objectives of the project are to:

- Develop an expandable DC microgrid model using MATLAB/Simulink and Simscape Electrical.
- Study the behavior of DC sources, loads, converters, capacitors, and energy-storage systems.
- Model renewable energy sources and connect them to a common DC bus.
- Apply local control strategies to individual source subsystems.
- Implement MPPT control for photovoltaic generation.
- Use battery energy storage to support DC bus voltage regulation.
- Monitor voltage, current, power, battery charge, and controller behavior.
- Study source/load disturbances and system response.
- Build toward a multi-source renewable DC microgrid containing PV, battery, and wind generation.

---

## Development Path

The model was developed incrementally so that each electrical and control concept could be tested before system-level integration.

```text
Basic DC Source + Resistive Load
                │
                ▼
        Variable Load Test
                │
                ▼
      Variable Source Tests
                │
                ▼
   PV-Like Controlled Current Source
                │
                ▼
        DC Bus Capacitor Test
                │
                ▼
       DC-DC Converter Test
                │
                ▼
        Actual PV Subsystem
        + Buck-Boost Converter
                │
                ▼
        P&O MPPT Controller
                │
                ▼
        Battery Subsystem
        + Buck-Boost Converter
                │
                ▼
 Closed-Loop DC Bus Voltage Control
                │
                ▼
      Battery Load-Change Test
                │
                ▼
       PV + Battery Integration
                │
                ▼
 Centralized Signal Monitoring
                │
                ▼
       CURRENT PROJECT STATE
                │
                ▼
        Wind Subsystem [Planned]
                │
                ▼
   Expanded DC Microgrid [Future]
```

This step-by-step approach made it possible to validate individual components before combining them into the larger microgrid.

---

## Current System Architecture

```text
                           ┌─────────────────────┐
                           │    PV Subsystem     │
                           │                     │
                           │ PV Source           │
                           │ P&O MPPT Controller │
                           │ Buck-Boost Converter│
                           └──────────┬──────────┘
                                      │
                                      │
                           ┌──────────▼──────────┐
                           │                     │
                           │    Common DC Bus    │
                           │       ~50 V         │
                           │                     │
                           │  Bus Capacitor      │
                           │  Load               │
                           │  V / I / P Sensors  │
                           │                     │
                           └──────────▲──────────┘
                                      │
                                      │
                         ┌────────────┴────────────┐
                         │    Battery Subsystem    │
                         │                         │
                         │ 36 V Battery            │
                         │ Buck-Boost Converter    │
                         │ Bus Voltage Controller  │
                         │ Charge Monitoring       │
                         └─────────────────────────┘


                      Future Expansion

                         Wind Subsystem
                               │
                               ▼
                        Common DC Bus
```

---

## Integrated DC Microgrid Model

The PV and battery subsystems are connected in parallel to the same DC bus.

The PV subsystem is responsible for extracting the maximum available solar power, while the battery subsystem supplies the remaining power required by the load and helps regulate the DC bus voltage.

![Integrated DC Microgrid Model](media/Integrated%20Model%20Schematic.png)

---

## PV Subsystem

The PV subsystem was developed using a Simscape photovoltaic source with an irradiance input and an Average-Value DC-DC Converter configured for buck-boost operation.

During the initial subsystem test, the converter was controlled using a fixed duty cycle. This verified that the PV source, converter, measurement blocks, and DC connection were operating correctly.

A local MPPT controller was then implemented to automatically adjust the converter duty cycle and move the PV operating point toward maximum available power.

### P&O MPPT Control

The MPPT controller uses measured PV voltage and current to calculate:

```text
Ppv = Vpv × Ipv
```

A Perturb and Observe algorithm then slightly changes the converter duty cycle and observes the resulting change in PV power.

If power increases, the algorithm continues perturbing in the same direction. If power decreases, the perturbation direction is reversed.

Zero-Order Hold, Saturation, and Unit Delay blocks were incorporated into the control path for sampled control behavior, duty-cycle limiting, and algebraic-loop prevention.

The MPPT controller increased PV power from approximately **119 W with fixed-duty operation to about 203.6 W** under the tested operating conditions.

![PV Subsystem with MPPT Controller](media/PV%20Subsystem%20with%20Local%20MPPT%20Controller.png)

---

## Battery Subsystem

The battery subsystem was developed as a separate energy-storage branch.

The modeled battery uses:

- Nominal voltage: **36 V**
- Capacity: **10 Ah**
- Initial charge: approximately **8 Ah / 80%**
- Buck-boost DC-DC converter
- Voltage and current measurement
- Battery power calculation
- Battery charge monitoring (`Q_bat`)
- Local DC bus voltage controller

Because the battery voltage is below the approximately 50 V DC bus voltage during discharge, the converter operates primarily in boost behavior under the tested conditions.

### Closed-Loop Bus Voltage Control

Unlike the PV subsystem, the battery is not controlled for maximum power extraction.

Its controller measures the actual DC bus voltage and compares it with a reference:

```text
Voltage Error = Vref - Vbus
```

where:

```text
Vref = 50 V
```

The controller adjusts the battery converter duty cycle based on this error.

The feedback loop therefore behaves conceptually as:

```text
50 V Reference
      │
      ▼
Compare with Measured Vbus
      │
      ▼
 Voltage Error
      │
      ▼
 Controller
      │
      ▼
Converter Duty Cycle
      │
      ▼
Battery Power Contribution
      │
      ▼
 DC Bus Voltage
      │
      └────────── Feedback ──────────┘
```

The controller was tuned experimentally until stable bus-voltage regulation was achieved.

![Battery Subsystem with Local Controller](media/Battery%20Subsystem%20with%20Local%20Controller.png)

---

## Battery Load-Change Testing

The battery subsystem was also tested under changing load demand.

The load resistance was changed from:

```text
10 Ω → 7.5 Ω
```

at approximately:

```text
t = 1 s
```

The increased load required more current and power from the battery.

After the load change, approximately:

| Quantity | Result |
|---|---:|
| DC Bus Voltage | 50.21 V |
| Load Current | 6.695 A |
| Load Power | 336.2 W |
| Battery Current | 10.04 A |
| Battery Power | 336.2 W |

The battery controller increased its power contribution while maintaining the DC bus close to the 50 V reference.

Battery charge also decreased faster after the disturbance, confirming the higher battery discharge rate.

---

## PV + Battery Integration

After both subsystems were validated independently, they were connected in parallel to the same DC bus.

For the initial integrated test:

- PV irradiance was maintained at **1000 W/m²**
- Load resistance was **10 Ω**
- The PV subsystem operated using MPPT
- The battery subsystem regulated the DC bus voltage

### Integrated Test Results

| Parameter | Approximate Result |
|---|---:|
| PV Voltage | 29.89 V |
| PV Current | 6.817 A |
| **PV Power** | **203.8 W** |
| Battery Voltage | 34.32 V |
| Battery Current | 1.744 A |
| **Battery Power** | **59.86 W** |
| DC Bus Voltage | **51.34 V** |
| Load Current | 5.135 A |
| **Load Power** | **263.6 W** |

The measured power balance was approximately:

```text
PV Power + Battery Power ≈ Load Power

203.8 W + 59.86 W ≈ 263.6 W
```

This demonstrated successful source integration and power sharing.

The PV subsystem supplied most of the available power, while the battery supplied the remaining demand and supported DC bus voltage regulation.

---

## Monitoring

As the model became larger, measurement wires and scopes began making the main schematic difficult to read.

To improve organization, Simulink **Goto/From** signal routing was used to create a dedicated monitoring area.

Signals currently monitored include:

### PV

- `PV_V` — PV voltage
- `PV_I` — PV current
- `PV_P` — PV power
- `PV_D` — MPPT duty cycle

### Battery

- `BAT_V` — battery voltage
- `BAT_I` — battery current
- `BAT_P` — battery power
- `BAT_Q` — battery charge
- `BAT_D` — battery converter duty cycle

### DC Bus

- `BUS_V` — DC bus voltage
- `BUS_I` — load/bus current
- `BUS_P` — load/bus power

Displays are primarily used for steady-state values, while scopes are used to study dynamic behavior.

---

## Key Engineering Concepts Applied

This project has involved practical application of:

- DC microgrid modeling
- MATLAB/Simulink
- Simscape Electrical
- Power electronics
- DC-DC conversion
- Buck-boost converters
- Photovoltaic source modeling
- Renewable energy integration
- Perturb and Observe MPPT
- Battery energy storage
- Closed-loop feedback control
- DC bus voltage regulation
- Controller tuning
- Power balance
- Load disturbance testing
- Voltage/current/power measurement
- System integration
- Subsystem-based modeling
- Signal monitoring and model organization

---

## Repository Structure

```text
Dc-Microgrid-Research/
│
├── README.md
│
├── docs/
│   └── Project documentation and research notes
│
├── models/
│   └── Simulink / Simscape model files
│
├── code/
│   └── MATLAB functions and controller code
│
└── media/
    ├── Integrated Model Schematic.png
    ├── PV Subsystem with Local MPPT Controller.png
    └── Battery Subsystem with Local Controller.png
```

The repository is being updated as the project progresses. Additional Simulink models and MATLAB controller files will be added as the project files are consolidated.

Detailed development documentation, test descriptions, and additional simulation results are available in the [`docs`](docs/) directory.

---

## Current Status

### Completed

- [x] Microgrid fundamentals and architecture study
- [x] Basic DC source/load simulations
- [x] Variable load testing
- [x] Variable source testing
- [x] DC bus capacitor testing
- [x] DC-DC converter testing
- [x] PV subsystem development
- [x] Buck-boost converter integration
- [x] P&O MPPT controller
- [x] Battery subsystem development
- [x] Closed-loop DC bus voltage control
- [x] Battery load-change testing
- [x] Centralized monitoring structure
- [x] PV + battery integration
- [x] Initial system-level power-sharing verification

### Planned

- [ ] Integrated variable-irradiance testing
- [ ] Improved battery charge/discharge control
- [ ] Battery SOC protection logic
- [ ] Automatic battery operating-mode control
- [ ] Wind-generation subsystem
- [ ] Wind subsystem control
- [ ] PV + battery + wind integration
- [ ] Additional disturbance and operating-condition testing
- [ ] Expanded system monitoring

### Possible Longer-Term Extensions

Depending on the direction of the research, later work may include:

- Energy-management strategies
- Communication-system simulation
- Operator dashboard / GUI
- Communication latency and packet-loss analysis
- Fault or abnormal-behavior monitoring
- AC microgrid modeling
- Hybrid AC/DC microgrid development

---

## Project Scope

This repository currently represents a **simulation-based research and modeling project**. The electrical system, converters, renewable sources, battery, and controllers are modeled in MATLAB/Simulink and Simscape Electrical rather than implemented as physical microgrid hardware.

The goal is to first establish a working and understandable DC microgrid model before expanding the system to additional energy sources and more advanced control, monitoring, and communication functions.

---

## Documentation

The [`docs`](docs/) directory contains the more detailed project documentation, including the learning process, individual simulation tests, subsystem development, controller implementation, and integration results.

---

## Project Status

**Ongoing — PV and battery integration complete. Wind subsystem development planned next.**
