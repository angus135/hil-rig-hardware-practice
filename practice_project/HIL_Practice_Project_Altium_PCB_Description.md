# Practice Project (Altium Designer): HIL Interface + Protection + IO Expansion Board

## 1. Project overview

### Goal
Design a **realistic, medium-complexity PCB** in **Altium Designer** that bridges a test system (your HIL rig / Raspberry Pi / MCU dev board) to an external **Device Under Test (DUT)** using robust connectors, protected IO, and clean power distribution.

This is intentionally shaped like a “real engineering board” rather than a toy board:
- it touches external wiring (ESD/protection matters),
- it uses real power rails (power tree matters),
- it has mixed-signal IO (layout matters),
- it produces real manufacturing outputs (deliverables matter).

### What you will learn / practice
This project is meant to force you to practice the parts of PCB design that people usually skip in tutorials:
- component + footprint selection (and library hygiene)
- input power protection
- switching regulator + LDO design
- interface transceivers (CAN / RS485)
- connector design and pinout documentation
- design rules, net classes, differential pairs (if used)
- decoupling and layout discipline
- DFM outputs (gerbers, drill, BOM, pick-and-place)
- documentation and review process

### Expected effort
- **20–35 hours** for a typical beginner-to-intermediate student team member
- can be done as an individual board project
- a “good enough” design can be completed in ~24 hours if you scope carefully

---

## 2. System context

### System role
This PCB sits between:
- **HIL/Test Controller Side**: a Raspberry Pi or MCU-based controller that runs automated tests
- **DUT Side**: the device being tested (another PCB / embedded system)

The board provides:
- regulated power rails (5 V and 3.3 V)
- protected harness IO for digital/analog signals
- at least one external comms interface (CAN or RS485)
- test points + status LEDs
- documentation and manufacturing files

---

## 3. Functional requirements

### 3.1 Input power

**Input voltage source:**
- The board must accept **12 V DC input**
- Connector options:
  - 2-pin screw terminal **OR**
  - barrel jack (2.1 mm recommended)
- Provide a clear label on silkscreen: `VIN_12V`

**Required protection & conditioning:**
- Reverse polarity protection:
  - preferred: ideal diode controller OR P-channel MOSFET reverse protection
  - acceptable: series Schottky diode (less preferred due to loss)
- Overcurrent protection:
  - required: fuse or resettable polyfuse on VIN
- TVS diode:
  - required on VIN to clamp spikes
- Input filtering:
  - at least one bulk capacitor on VIN near connector
  - optionally a pi-filter (not required)

**Indicators:**
- Power LED indicating VIN present
- Must not draw excessive current (use resistor sizing appropriately)

---

### 3.2 Power tree

The board must generate the following rails:

**5 V rail**
- generated from VIN (12 V) by **a switching buck regulator**
- minimum output current: **1 A**
- must include:
  - inductor
  - input/output caps per datasheet
  - feedback resistors
  - enable pin handling (see below)
- rail name: `+5V`

**3.3 V rail**
- generated from 5 V by **an LDO**
- minimum output current: **500 mA**
- provide `+3V3` net
- must include:
  - required input/output capacitance per datasheet
  - placement close to regulator pins

**Regulator control**
- Provide at least one of:
  - an **ENABLE jumper** for 5 V buck
  - a **power-good signal** LED
  - a **power sequencing** statement in docs (even if trivial)

**Documentation requirement**
- A short “Power Tree” diagram in the documentation section (can be ASCII or embedded in a PDF later)

---

### 3.3 Control / logic domain

This board must include **one logic/control block**:

Choose **Option A** or **Option B**.

#### Option A (more realistic): Microcontroller
- Include a microcontroller such as:
  - STM32 (any small package acceptable)
  - RP2040
  - ATSAMD
- Requirements:
  - SWD or programming header
  - reset button
  - boot configuration (if applicable)
  - a UART debug header (TX/RX/GND)
  - at least one status LED driven by firmware

#### Option B (simpler, still useful): IO expanders
- Use one or more GPIO expanders via I2C, such as:
  - MCP23017 (16 GPIO)
  - PCA9555, etc.
- Requirements:
  - include I2C pullups (confirm values)
  - include address selection jumpers or solder bridges
  - include a dedicated interrupt line routed to a header pin

> Recommendation: Option B is ideal if you want the focus on PCB design rather than firmware.

---

### 3.4 External comms interface (must implement at least one)

Choose one:
- **CAN** (highly recommended for motorsport / embedded testing)
- **RS485**

#### CAN requirements
- CAN transceiver + connector
- Connector options:
  - 3-pin header: CANH/CANL/GND
  - DB9 (nice but more mechanical work)
- Must include:
  - 120 Ω termination resistor selectable (jumper / switch / solder bridge)
  - ESD protection on CANH/CANL recommended
- Nets: `CANH`, `CANL`

#### RS485 requirements
- RS485 transceiver + connector
- Must include:
  - biasing/termination selectable
  - ESD protection recommended
- Nets: `A`, `B` (or `D+`, `D-` if you use that naming)

---

### 3.5 IO harness connectors (DUT-side signals)

The board must provide **two harness connectors** to simulate real IO wiring.

**Connector form factor**
- Choose one:
  - 2x 10-pin boxed header
  - JST-XH multi-pin connectors
  - screw terminals (bulkier)

**Minimum signal set**
Provide at least:

#### Digital outputs
- **8 outputs**
- must be protected and robust:
  - series resistor OR low-side switch/transistor stage
  - clamp diodes or ESD protection strongly recommended
- Each output must have:
  - label (e.g., `DO0..DO7`)
  - test point

#### Digital inputs
- **8 inputs**
- requirements:
  - input series resistor recommended
  - pullup/pulldown resistors (select a strategy and document)
  - protection against ±12 V miswire (at least clamp + resistor)
- Each input must have:
  - label (e.g., `DI0..DI7`)
  - test point

#### Analog inputs
- **2 analog inputs**
- must be scalable:
  - input is 0–12 V (or at least 0–5 V), **scaled to 0–3.3 V**
  - use resistor divider + optional RC filter
- Each analog input must have:
  - label (e.g., `AI0`, `AI1`)
  - test point

---

### 3.6 User interaction / observability

Include:
- at least **3 LEDs**
  - VIN present
  - 5 V present
  - 3.3 V present (or comm activity)
- at least **1 pushbutton**
  - reset or user button
- at least **10 test points**
  - include all key rails and at least some IO

---

## 4. PCB / layout requirements

### 4.1 Stackup
- **2-layer board**
- standard FR4
- 1 oz copper (default)

### 4.2 Board size & mechanical
- Max board outline: **100 mm x 100 mm**
- 4x mounting holes:
  - M3 preferred
  - keep copper clearance around holes
- Add board marking:
  - project name
  - revision: `REV A`
  - date
  - designer name/initials

### 4.3 Placement guidance (must follow)
- power input and power tree must be **clustered**
- keep switching regulator power loop compact
- keep analog input divider/filter away from switching node
- route external connectors with protection nearby

### 4.4 Design rules (must define)
Must define in Altium:
- net classes for:
  - power (VIN, 5 V, 3.3 V)
  - signals
- wider traces for:
  - VIN and 5 V rails
- clearance rules:
  - default 6/6 mil acceptable (or whatever your fab rules are)
- if using CAN/RS485:
  - define differential pair rules (even if loose)

---

## 5. Library & part selection rules

### 5.1 Allowed sources
- Use manufacturer datasheets as primary reference
- Footprints:
  - must be verified (no “random footprint” without checking)
  - verify:
    - pad size
    - pin 1 marking
    - courtyard
    - mechanical constraints

### 5.2 Mandatory checklist for each custom footprint
For each custom/added footprint you must document:
- source (datasheet page reference)
- dimensions verified (Y/N)
- 3D model (optional but encouraged)

---

## 6. Deliverables

### 6.1 Design files (Altium)
- `.PrjPcb`
- schematic(s)
- PCB layout
- libraries used (integrated library recommended)

### 6.2 Manufacturing outputs
- gerbers
- drill files
- board outline included
- generated with:
  - correct units
  - correct layer naming
  - correct solder mask / paste layers

### 6.3 Assembly outputs
- BOM with:
  - Manufacturer Part Number (MPN)
  - quantity
  - designator references
- Pick-and-place file (centroid)
- Optional: assembly drawings (PDF)

### 6.4 Documentation
Provide a short project report (can be in markdown or pdf) that includes:
- system overview
- power tree description
- connector pinout tables
- notable design decisions (e.g., termination strategy)

---

## 7. Testing & review

### 7.1 Design review checklist (minimum)
Before marking as complete, confirm:
- no unrouted nets
- ERC/DRC clean (or justify each violation)
- silkscreen readable and correct
- pin 1 clearly marked on ICs and connectors
- all power rails labelled
- important nets have test points
- termination jumpers clearly labelled

### 7.2 “Student misuse” test scenario
You must address, at least in documentation, what happens if:
- VIN reverse polarity
- IO shorted to GND
- IO shorted to VIN
- ESD strike on connector pin

---

## 8. Marking rubric (suggested)

| Area | Weight |
|---|---:|
| Schematic correctness & clarity | 25% |
| Power design correctness | 20% |
| IO protection & robustness | 20% |
| PCB layout quality | 20% |
| Deliverables quality (BOM/PNP/Gerbers/docs) | 15% |

---

## 9. Stretch goals (optional)

If you finish early, implement one or more:
- replace screw terminals with locking connectors
- add current sensing on VIN (shunt + amplifier)
- add EEPROM for board ID
- add isolated RS485 or CAN
- add “DUT selectable power” (high-side switch)
- add analog output (DAC + buffer) for stimulus generation

---

## 10. Final note
This practice board should be treated like a real product:
- keep the schematic readable
- make connectors clearly labelled
- assume someone else will wire it without asking you questions
- aim for a board that would realistically survive student lab conditions
