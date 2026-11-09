# Programmable Motorized XY Linear Stage for Optical Fibre Tapering and Torsion

A programmable, motorized platform for **optical-fibre tapering and
torsion**, developed at the **National Institute of Technology Patna**
during the 2025--2026 academic year.

The prototype uses an Arduino UNO to command four DM542T stepper-motor
drivers and four NEMA 17 stepper motors. Two motors move opposing linear
stages for fibre pulling, while a third motor provides controlled
rotation for fibre torsion. A heating torch softens the fibre during
tapering.

> **Implementation note:** The project report documents an Arduino UNO /
> ATmega328P controller and Arduino firmware. It does **not** document
> an FPGA implementation or existing Verilog RTL. This README describes
> the reported hardware faithfully and includes a clearly separated
> roadmap for a possible Verilog/FPGA implementation. It does not claim
> that such RTL has already been developed or tested.

------------------------------------------------------------------------

## Table of Contents

-   [Project Overview](#project-overview)
-   [Objectives](#objectives)
-   [How the System Works](#how-the-system-works)
-   [System Architecture](#system-architecture)
-   [Hardware and Bill of Materials](#hardware-and-bill-of-materials)
-   [Motor Allocation](#motor-allocation)
-   [Arduino Pin Mapping](#arduino-pin-mapping)
-   [Electrical Interface](#electrical-interface)
-   [Motion Resolution](#motion-resolution)
-   [DM542T Configuration](#dm542t-configuration)
-   [Mechanical Arrangement](#mechanical-arrangement)
-   [Operating Procedure](#operating-procedure)
-   [Reported Results](#reported-results)
-   [Limitations](#limitations)
-   [Future Work: FPGA and Verilog
    RTL](#future-work-fpga-and-verilog-rtl)
-   [Suggested Repository Structure](#suggested-repository-structure)
-   [Development and Reproduction](#development-and-reproduction)
-   [Safety Notes](#safety-notes)
-   [Project Information](#project-information)
-   [References](#references)

------------------------------------------------------------------------

## Project Overview

Optical fibres are often modified to obtain useful optical and sensing
characteristics. This project develops a low-cost motorized setup for
two such processes:

1.  **Fibre tapering:** A central section of the optical fibre is heated
    and pulled so that its diameter decreases, forming a thin taper
    waist.
2.  **Fibre torsion:** One end of the fibre is rotated while the other
    end is held fixed, applying a controlled twist around the fibre
    axis.

Manual pulling and twisting can be difficult to reproduce consistently.
The developed platform uses stepper motors, digital motor drivers and
programmable control signals to make the mechanical movement repeatable.

The assembly is mounted on an optical breadboard. Two precision linear
stages face one another and hold the fibre along a common axis. The
heating torch is positioned at the centre of the fibre span.

## Objectives

-   Build a motorized platform for optical-fibre tapering and torsion.
-   Pull both ends of a heated fibre in a controlled manner.
-   Generate controlled fibre rotation using a separate motor.
-   Use programmable step and direction signals to control motor
    movement.
-   Improve repeatability compared with manual movement.
-   Provide a cost-effective platform for photonics laboratory
    experiments.

## How the System Works

### 1. Fibre tapering

The fibre is stripped and clamped between two holders mounted on
opposing linear stages. A heating torch softens the central region.
Motors **M1** and **M2** then move the holders outward, stretching the
heated section.

The central region becomes thinner and forms the taper waist. The report
describes the torch as manually positioned and activated; it is not
electronically temperature-controlled.

### 2. Fibre torsion

One fibre holder is coupled to motor **M3**. The other end is held
fixed. Rotating M3 twists the fibre around its longitudinal axis. The
number of commanded motor steps determines the commanded angular
displacement, subject to the motor-driver microstep setting and the
mechanical coupling.

Torsion may be applied during or after the tapering sequence, as
described in the report.

------------------------------------------------------------------------

## System Architecture

The system can be understood as three functional layers:

  -----------------------------------------------------------------------
  Layer                   Main elements           Responsibility
  ----------------------- ----------------------- -----------------------
  Control                 Arduino UNO             Generates STEP/PUL and
                                                  DIR signals

  Drive                   Four DM542T drivers and Converts control inputs
                          24 V DC supply          into motor phase
                                                  current

  Motion                  Four NEMA 17 motors and Produces linear and
                          mechanical stages       rotational movement
  -----------------------------------------------------------------------

### Functional signal path

``` text
                 Arduino UNO
              (ATmega328P, 5 V)
                      |
             STEP/PUL and DIR
                      |
          +-----------+-----------+
          |           |           |
       DM542T M1   DM542T M2   DM542T M3   DM542T M4
          |           |           |           |
       NEMA 17     NEMA 17     NEMA 17     NEMA 17
          |           |           |           |
      Left stage  Right stage   Torsion    Auxiliary
        (pull)      (pull)       axis        axis

       24 V DC supply powers the DM542T driver power inputs.
```

The Arduino provides low-voltage control signals. The DM542T drivers
supply the current required by the stepper-motor windings. The Arduino
is not intended to power the motors directly.

## Hardware and Bill of Materials

The following quantities and specifications are taken from the project
report.

  ------------------------------------------------------------------------
  Component                                 Quantity Reported
                                                     specification / role
  --------------------- ---------------------------- ---------------------
  Arduino UNO                                      1 ATmega328P, 5 V
                                                     logic; main
                                                     controller

  DM542T digital                                   4 20--50 V DC operating
  stepper driver                                     range; adjustable
                                                     1.0--4.2 A
                                                     output-current range

  NEMA 17 stepper motor                            4 1.8° full-step angle;
                                                     200 full
                                                     steps/revolution

  Precision linear                                 2 Lead screw and guide
  stage                                              rails

  DC power supply                                  1 24 V DC, 10 A or
                                                     higher recommended

  Heating torch / gun                              1 Manually positioned
                                                     heat source for fibre
                                                     softening

  Optical breadboard                               1 Rigid mounting
                                                     platform with M6
                                                     threaded holes

  Fibre holders                                    2 Clamp the fibre to
                                                     the moving stages
  ------------------------------------------------------------------------

The report gives a typical NEMA 17 holding-torque range of 0.3--0.6 Nm
and a rated phase-current range of 1.5--2.0 A. Actual motor and driver
settings must be checked against the nameplate and manufacturer
documentation for the specific hardware used.

## Motor Allocation

  -------------------------------------------------------------------------
  Motor               Arduino STEP pin      Arduino DIR pin Function
  --------------- -------------------- -------------------- ---------------
  M1                                D2                   D3 Left linear
                                                            stage; moves
                                                            the left fibre
                                                            holder outward

  M2                                D4                   D5 Right linear
                                                            stage; moves
                                                            the right fibre
                                                            holder outward

  M3                                D6                   D9 Torsion motor;
                                                            rotates one
                                                            fibre holder

  M4                                D7                   D8 Auxiliary
                                                            motor; reserved
                                                            for future use
                                                            or additional
                                                            functions
  -------------------------------------------------------------------------

For symmetric taper pulling, M1 and M2 are commanded to move outward
together by corresponding amounts. The report describes M4 as auxiliary;
it does not assign M4 a required role in the core tapering/torsion
operation.

## Arduino Pin Mapping

The report specifies the following Arduino UNO digital-pin assignments:

  -----------------------------------------------------------------------
  Arduino UNO pin   Signal            Driver input      Purpose
  ----------------- ----------------- ----------------- -----------------
  D2                PUL M1            M1 PUL+           Left-stage step
                                                        pulses

  D3                DIR M1            M1 DIR+           Left-stage
                                                        direction

  D4                PUL M2            M2 PUL+           Right-stage step
                                                        pulses

  D5                DIR M2            M2 DIR+           Right-stage
                                                        direction

  D6                PUL M3            M3 PUL+           Torsion step
                                                        pulses

  D9                DIR M3            M3 DIR+           Torsion direction

  D7                PUL M4            M4 PUL+           Auxiliary step
                                                        pulses

  D8                DIR M4            M4 DIR+           Auxiliary
                                                        direction

  GND               Common ground     Signal returns as Shared signal
                                      shown in the      reference
                                      report            
  -----------------------------------------------------------------------

The report describes the Arduino STEP/PUL and DIR signals as being
connected to the corresponding positive signal terminals, with the
negative signal terminals connected to common ground. Confirm the exact
input wiring and current requirements against the manual for the
specific DM542T version before powering the system.

## Electrical Interface

Each DM542T has a signal-input side and a motor-power/output side.

  DM542T terminal   Function described in the report
  ----------------- ------------------------------------
  PUL+ / PUL−       Step-pulse input
  DIR+ / DIR−       Direction input
  ENA+ / ENA−       Driver-enable input
  VDC               Positive motor-driver supply input
  GND               Negative motor-driver supply input
  A+ / A−           Motor phase-A winding
  B+ / B−           Motor phase-B winding

### Power distribution

-   The 24 V DC positive rail is distributed to the VDC input of each
    driver.
-   The supply negative rail is connected to the GND input of each
    driver.
-   The report specifies a common ground connection between the power
    supply and Arduino.
-   Each driver's A+/A− and B+/B− outputs connect to the corresponding
    motor windings.

### Important wiring qualification

The report states that ENA− is kept LOW to keep the driver enabled, but
it does not provide a complete ENA+ wiring detail. The exact
enable-terminal arrangement, signal polarity, input-current requirements
and acceptable pulse timing must be verified using the manufacturer's
manual for the actual driver. Do not assume that every DM542T clone or
revision has identical input wiring.

The report's wiring description is a project-level summary, not a
substitute for the driver manufacturer's wiring and safety
documentation.

## Motion Resolution

The report uses the following values for its stated linear-resolution
calculation:

-   Motor full-step angle: **1.8°**
-   Full steps per revolution: **200**
-   Driver microstep setting: **1600 steps/revolution**
-   Linear-stage lead-screw travel: **1 mm/revolution**

Therefore:

``` text
Linear travel per microstep
    = Lead-screw travel per revolution
      / Driver steps per revolution

    = 1 mm / 1600
    = 0.000625 mm
    = 0.625 µm
```

The reported nominal commanded linear increment is therefore **0.625 µm
per microstep**.

This is a theoretical command resolution derived from the stated
microstep and lead-screw values. It should not be interpreted as
verified absolute positioning accuracy. Actual movement can be affected
by mechanical backlash, alignment, friction, motor torque, load, missed
steps and microstepping nonlinearity.

For a configured stage with lead-screw pitch `P` (in mm/revolution) and
driver resolution `S` (in pulses/revolution):

``` text
Commanded travel (mm) = Number of pulses × P / S

Required pulses       = Desired travel (mm) × S / P
```

These equations assume the motor is coupled directly to the lead screw
with no additional gear ratio.

## DM542T Configuration

The report specifies these settings for each driver:

  Setting                Reported value
  ---------------------- -----------------------
  Motor current          2.0 A
  Microstep resolution   1600 steps/revolution
  Driver supply          24 V DC

The report identifies SW1--SW3 as the current-setting switches and
SW5--SW8 as the microstep-setting switches. It does not provide a
complete switch-by-switch ON/OFF table. Use the DIP-switch table in the
manual for the exact driver model and revision; do not infer switch
positions from the resolution alone.

Set the current to a value compatible with the actual motor's rated
phase current and the driver's specified current convention (peak or
RMS).

------------------------------------------------------------------------

## Mechanical Arrangement

-   Two linear stages are mounted symmetrically on the optical
    breadboard.
-   Their axes should be collinear and face toward the central heating
    region.
-   Each moving platform carries a fibre holder.
-   M1 drives the left stage; M2 drives the right stage.
-   The heating torch is manually positioned below the fibre midpoint.
-   M3 is coupled to one fibre holder for torsion.
-   The Arduino, drivers and power supply are mounted away from the
    moving and heating region.

Mechanical alignment is important: misaligned stages can bend the fibre,
introduce uneven loading or affect the taper profile.

## Operating Procedure

The following sequence summarizes the procedure in the report. It is
intended for trained laboratory users following local laboratory safety
procedures.

### Phase 1 --- Assembly and setup

1.  Gather the Arduino UNO, four DM542T drivers, four NEMA 17 motors,
    two linear stages, power supply, heating torch and mounting
    hardware.
2.  Mount the motors to the stages and secure both stages to the optical
    breadboard.
3.  Align the stages on a common axis and attach the fibre holders.
4.  Complete the signal, power and motor-winding connections.
5.  Configure the driver current and microstep settings for the actual
    motors.
6.  Upload the Arduino control program using the Arduino IDE.

### Phase 2 --- Test without fibre

1.  Keep the heating torch off.
2.  Power the motor-driver supply and check the driver and Arduino
    status indicators.
3.  Test each motor individually.
4.  Confirm direction, movement and speed.
5.  Check that both linear stages move smoothly without binding or
    wobble.
6.  Verify that the two pulling stages move outward in the intended
    directions before installing a fibre.

### Phase 3 --- Fibre tapering and optional torsion

1.  Strip approximately 3 cm of coating from the central fibre section,
    as specified in the report.
2.  Clean the stripped section carefully with isopropanol and allow it
    to evaporate fully before heating.
3.  Clamp both ends in the fibre holders, keeping the fibre straight and
    free of kinks.
4.  Position the heating torch below the central stripped section.
5.  Apply heat and run the programmed outward movement of M1 and M2.
6.  If required, command M3 to apply torsion during or after pulling.
7.  Stop the heating process and allow the fibre to cool before
    handling.
8.  Remove and inspect the fibre under a microscope for taper
    uniformity, shape and surface quality.

The report describes an approximately 5-second softening interval and
approximately 30-second cooling interval. These are reported procedure
values, not universal process settings; suitable heating and cooling
depend on the fibre, flame, geometry and laboratory conditions.

------------------------------------------------------------------------

## Reported Results

The project report states that the prototype achieved:

-   **Programmable motion:** Motor speed was varied by changing pulse
    delay in the Arduino program.
-   **Fine commanded motion:** A nominal 0.625 µm stage increment was
    calculated from the stated 1 mm/revolution lead screw and 1600
    driver steps/revolution.
-   **Repeatability:** Fixed step counts were used to reproduce
    commanded motion across trials.
-   **Combined operation:** M1 and M2 were operated for pulling, with M3
    providing torsion in an automated sequence.
-   **Low-cost modular construction:** Standard Arduino, driver and
    stepper-motor components were used.

These are results as reported by the project document. The report does
not provide a quantitative measurement dataset for taper-diameter error,
positioning accuracy, repeatability statistics, optical loss or
torsion-angle calibration. Such values should be added only after they
have been measured and documented.

## Limitations

The report identifies the following limitations:

1.  **Open-loop motion:** No encoder or position-feedback sensor is
    used. The controller commands steps but does not independently
    verify actual position.
2.  **Manual heating:** Torch positioning and heating are not
    automatically controlled.
3.  **No graphical interface:** Parameters are changed in the Arduino
    program and uploaded again.
4.  **No real-time fibre monitoring:** The system does not measure taper
    diameter during the process.

## Future Work: FPGA and Verilog RTL

The current project is Arduino-based. If the control system is later
migrated to an FPGA, a possible RTL architecture could separate pulse
generation, direction control, sequencing and parameter handling into
independent modules.

A conceptual module breakdown is:

  -----------------------------------------------------------------------
  Potential RTL module                Responsibility
  ----------------------------------- -----------------------------------
  `step_pulse_gen`                    Generate configurable STEP pulses

  `direction_ctrl`                    Set motor direction

  `motor_axis_ctrl`                   Combine pulse and direction control
                                      for one axis

  `dual_stage_sync`                   Coordinate M1 and M2 for symmetric
                                      pulling

  `torsion_ctrl`                      Generate the M3 rotation command

  `motion_sequencer`                  Sequence setup, pull, torsion and
                                      stop operations

  `parameter_regs`                    Store pulse count, speed and
                                      direction settings

  `top`                               Integrate the control modules and
                                      expose hardware I/O
  -----------------------------------------------------------------------

### Suggested FPGA development stages

1.  Define the FPGA board, clock frequency, I/O voltage standards and
    driver-interface electrical requirements.
2.  Specify the timing requirements of the DM542T STEP, DIR and enable
    inputs from the exact driver manual.
3.  Implement a parameterized pulse generator in synthesizable Verilog.
4.  Implement independent axis controllers and a coordinated M1/M2
    motion sequencer.
5.  Add M3 torsion control and a safe stop/state-control mechanism.
6.  Verify pulse count, pulse width, direction setup/hold time and
    simultaneous-axis behavior in simulation.
7.  Test the FPGA outputs with suitable interface circuitry before
    connecting motor drivers.
8.  Add limit switches, emergency-stop handling and position feedback
    where the hardware design supports them.

**No Verilog source, FPGA pin constraints, simulation testbench,
synthesis result or FPGA timing report is included in the project
report.** These should be added to the repository only when implemented
and verified. The Arduino pin mapping above is not an FPGA pin
assignment.

------------------------------------------------------------------------

## Suggested Repository Structure

The following is a suggested organization for a GitHub repository. It is
not a claim that these files were included in the submitted project
report.

``` text
optical-fibre-tapering-torsion/
├── README.md
├── docs/
│   ├── project-report.pdf
│   ├── wiring-diagram.png
│   └── prototype.jpg
├── arduino/
│   ├── optical_fibre_controller/
│   │   └── optical_fibre_controller.ino
│   └── README.md
├── hardware/
│   ├── bill-of-materials.md
│   ├── pin-mapping.md
│   └── wiring/
├── fpga/                       # Future work; add when RTL exists
│   ├── rtl/
│   ├── testbench/
│   ├── constraints/
│   └── README.md
└── LICENSE
```

Rename or remove placeholder paths to match the files actually committed
to the repository. Do not add empty or misleading RTL folders if the
project remains Arduino-only.

## Development and Reproduction

### Arduino implementation

The report specifies use of the Arduino IDE and an Arduino UNO.

1.  Install the Arduino IDE.
2.  Connect the Arduino UNO to the development computer over USB.
3.  Open the project's `.ino` control program.
4.  Select the correct Arduino UNO board and serial port.
5.  Review the pin mapping and motor parameters.
6.  Compile and upload the program.
7.  Test motor operation without a fibre or active heating source before
    experimental use.

**Firmware availability:** The report explains the control arrangement
and operating procedure, but does not include the actual Arduino source
code in the document. Add the verified `.ino` file to `arduino/` before
presenting this repository as directly buildable.

### FPGA implementation

There is no FPGA build flow documented in the report. An FPGA
implementation will need a selected device, toolchain, top-level pin
constraints, timing constraints and verified RTL/testbenches before
reproducible build instructions can be provided.

## Safety Notes

-   Disconnect motor power before changing motor, driver or supply
    wiring.
-   Verify supply polarity and driver terminal labels before energizing
    the system.
-   Set driver current according to the actual motor and driver
    documentation.
-   Do not connect a stepper motor directly to Arduino I/O pins.
-   Keep hands, cables and loose objects clear of moving stages and
    couplings.
-   Use appropriate eye protection, ventilation, fire precautions and
    laboratory supervision when operating a heating torch.
-   Ensure isopropanol has fully evaporated before bringing the fibre
    near a flame.
-   Allow the fibre and nearby hardware to cool before handling.
-   Provide an accessible power-disconnect method. The report does not
    document a dedicated emergency-stop circuit; add one as a hardware
    safety improvement.
-   Treat the system as open-loop: a commanded step count does not prove
    that the stage reached its intended position.

This README is documentation, not a substitute for the DM542T
manufacturer's manual, electrical review or laboratory risk assessment.

## Project Information

  -----------------------------------------------------------------------
  Field                               Information
  ----------------------------------- -----------------------------------
  Institution                         National Institute of Technology
                                      Patna

  Department / domain                 Photonics and instrumentation
                                      project

  Project                             Programmable Motorized XY Linear
                                      Stage --- Optical Fibre Tapering
                                      and Torsion Device

  Academic year                       2025--2026

  Project supervisor                  Dr. Jitendra Bahadur Maurya

  Submitted by                        Shivam Maurya, Sourabh Pandey,
                                      Ayushi Kumari

  Main controller                     Arduino UNO (ATmega328P)

  Motor drivers                       4 × DM542T

  Motors                              4 × NEMA 17
  -----------------------------------------------------------------------

## References

The following references are listed in the project report:

1.  T. A. Birks and Y. W. Li, "The shape of fiber tapers," *Journal of
    Lightwave Technology*, vol. 10, no. 4, pp. 432--438, 1992.
2.  J. D. Love et al., "Tapered single-mode fibres and devices -- Part
    1: Adiabaticity criteria," *IEE Proceedings J -- Optoelectronics*,
    vol. 138, no. 5, pp. 343--354, 1991.
3.  B. E. A. Saleh and M. C. Teich, *Fundamentals of Photonics*, 3rd
    ed., Wiley, 2019.
4.  Arduino, *Arduino UNO Reference Manual*, 2024.
    https://www.arduino.cc
5.  Leadshine Technology Co., *DM542T Digital Stepper Drive User
    Manual*, version 1.0, 2020.
6.  X. Chen et al., "Twisted optical fiber sensors," *Sensors*, vol. 17,
    no. 5, p. 1168, 2017.

------------------------------------------------------------------------

*Documentation prepared from the project report "Programmable Motorized
XY Linear Stage --- Tapering and Torsion of Optical Fibre Device" (NIT
Patna, 2025--2026). Hardware details and reported results are attributed
to that report; proposed FPGA/Verilog work is explicitly identified as
future work.*
