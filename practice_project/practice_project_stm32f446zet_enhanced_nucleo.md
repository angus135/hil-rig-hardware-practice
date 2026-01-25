# Practice Project (Altium Designer): STM32F446ZET “Enhanced Nucleo” Development Board

## 1. Project overview

### Goal
Design a **realistic, medium-complexity STM32 development board** in **Altium Designer**, based around the **STM32F446ZET** (LQFP144), intended for student use.

This project is intentionally **more involved than a standard Nucleo board**, but still realistic and buildable:
- includes external memory,
- includes a real field bus interface (CAN + transceiver),
- supports multiple power sources,
- includes proper protection, test points, and documentation.

The output should feel like a board a student team could actually build, debug, and use in lab projects (and not a “toy PCB”).

### What you will learn / practice
This project forces practice of the parts that matter in real embedded boards:
- MCU hardware design (clocks, reset, boot strapping)
- SWD + ST-Link connector integration
- power-tree design (USB supply + battery supply + buck + LDO)
- external QSPI flash design + routing discipline
- CAN transceiver design (termination, ESD, connector pinout)
- practical protection (ESD, reverse polarity, fusing)
- design rules, net classes, controlled routing constraints (QSPI)
- DFM outputs (gerbers, drill, BOM, pick-and-place)
- documentation: pinouts, power tree, bring-up checklist

### Expected effort
- **20–40 hours** for a typical beginner-to-intermediate student
- can be completed as an individual board project
- a “good enough” board can be completed in ~24–30 hours if scope is controlled

---

## 2. System context

### System role
This PCB is a standalone MCU development board that provides:
- a robust STM32 platform for firmware development
- CAN communication via an onboard transceiver
- external flash for logging / firmware storage
- flexible powering from USB or battery
- headers compatible with breadboard/expansion wiring or a “shield” style adapter

It should explicitly be designed to survive student lab conditions:
- accidental shorts
- miswiring
- ESD from human handling
- hot-plugging connectors

---

## 3. Functional requirements

### 3.1 Core MCU subsystem (STM32F446ZET)
The board must use:
- **MCU:** STM32F446ZET (LQFP144)

Minimum MCU support circuitry:
- **VDD decoupling** per datasheet (multiple caps distributed across supply pins)
- **VCAP capacitors** (core regulator caps) per datasheet
- **NRST reset circuit**:
  - reset pushbutton
  - optional RC reset network (recommended)
- **BOOT0 configuration**:
  - BOOT0 pull-down by default
  - solder bridge or jumper to allow BOOT0 high (for system bootloader)
- **Clocking:**
  - include a **high speed external crystal (HSE)** (8–16 MHz)
  - optional: include 32.768 kHz LSE crystal footprint (can be DNI)

Observability:
- at least **1 user LED** connected to a GPIO pin
- at least **1 user button** connected to a GPIO pin (not NRST)

---

### 3.2 Debug / programming interface (ST-Link compatible)
The board must support external programming/debug via ST-Link.

Required:
- include an **SWD connector** (choose one option):
  - Option A: **ARM 2x5 1.27 mm SWD header** (recommended)
  - Option B: 2.54 mm 1x6/1x10 header (acceptable but less standard)

The SWD header must provide at minimum:
- `SWDIO`
- `SWCLK`
- `NRST`
- `GND`
- `VTREF` (3.3 V reference)
- optional: `SWO` (recommended)

Must include:
- silkscreen pin-1 marking
- short routing and clean return ground
- optional: ESD protection on external header pins (recommended)

---

### 3.3 USB interface (power + debug UART)
The board must include:
- **USB Micro-B or USB-C connector** (USB-C preferred for modernity)
- USB used for:
  - providing 5 V power
  - optionally a USB-to-UART bridge for serial debug

#### USB-to-UART requirements (recommended)
Include a USB-UART IC such as:
- CP2102N, CH340C, FT232, etc.

Must provide:
- 3-pin or 4-pin UART header: `TX`, `RX`, `GND`, optional `3V3`
- connect UART to an STM32 USART (document which)

Add:
- RX/TX activity LEDs (optional but encouraged)
- ESD protection on USB data lines (recommended)

---

### 3.4 Power input system (USB + battery)
The board must be powerable from:
1) **USB 5 V**
2) **External battery input** (nominal “2S” lithium style)

#### Battery input requirements
Battery input range:
- **6 V to 16 V** input range
- connector options:
  - 2-pin screw terminal
  - XT30 footprint (stretch goal)
  - JST-VH 2-pin (acceptable)

Protection required:
- reverse polarity protection:
  - preferred: ideal diode controller OR P-channel MOSFET reverse protection
  - acceptable: series Schottky diode (document efficiency loss)
- input fuse or polyfuse
- TVS diode on battery VIN
- bulk input capacitance near connector

Lab usability requirement:
- silkscreen label: `VBAT_IN`
- connector polarity clearly marked

---

### 3.5 Power tree (must include switching + LDO)
The board must generate:
- `+5V` system rail
- `+3V3` logic rail

#### 5 V rail
Requirements:
- board must support **USB 5 V** directly as a source of `+5V`
- board must also generate `+5V` from battery VIN using a **buck regulator**
- minimum output current capability: **1 A**

Source selection requirements (choose one approach and document it clearly):
- Option A: ideal diode OR-ing (preferred)
- Option B: power mux IC
- Option C: diode OR-ing (acceptable but lower efficiency)

#### 3.3 V rail
Requirements:
- generate `+3V3` from `+5V` using:
  - an **LDO** (minimum 500 mA), OR
  - a small buck regulator (stretch goal)

Must include:
- correct datasheet-required capacitors
- `+3V3` test point
- 3.3 V power LED

Regulator control:
- include at least one of:
  - enable jumper (buck enable)
  - power-good LED/signal

---

### 3.6 CAN interface (mandatory)
The board must provide a **CAN bus interface** using an onboard transceiver.

#### CAN physical layer
Must include:
- CAN transceiver (examples):
  - SN65HVD230
  - TJA1051
  - MCP2562
- `CANH` / `CANL` routed to an external connector

Connector options:
- 3-pin header `CANH`, `CANL`, `GND`
- OR DB9 footprint (recommended stretch goal)

Termination:
- 120 Ω termination resistor must be **selectable**
  - jumper, switch, or solder bridge
- termination clearly documented and labelled

Protection:
- ESD diodes on CANH/CANL recommended
- optional: common mode choke footprint (nice)

Indicators:
- optional CAN activity LED(s) driven by MCU (stretch)

---

### 3.7 External flash memory (mandatory, 4 MB)
The board must include:
- **4 MB external flash memory**
- preferred interface: **QSPI**
- acceptable interface: SPI (only if QSPI deemed too complex)

Requirements:
- 4 MB (32 Mbit) minimum capacity, QSPI recommended
- include proper decoupling
- include pull-ups/pull-downs as required by memory datasheet
- include series resistors (0–33 Ω footprints) for QSPI lines (recommended)

Nets must be clearly named (example):
- `QSPI_CLK`
- `QSPI_NCS`
- `QSPI_IO0..IO3`

Layout requirements for QSPI:
- keep traces short and matched “reasonably”
- define a net class for high speed signals
- avoid running QSPI over split ground

---

### 3.8 Expansion IO headers (Nucleo-style)
The board must include expansion headers so it can be used like a dev board.

Requirements:
- two large headers that expose:
  - power pins (`5V`, `3V3`, `GND`)
  - at least **20 GPIO pins**
  - I2C pins
  - SPI pins
  - at least one UART
  - at least 6 ADC-capable pins

Mechanical constraints:
- 2.54 mm headers
- keep-out for mounting holes
- silkscreen labels for each pin group


---

### 3.9 Board protection + lab robustness
Must include the following “student proofing” measures:
- polyfuse on USB 5 V input (recommended)
- battery input protection (reverse polarity + fuse/polyfuse + TVS)
- at least **2 ESD protection devices**:
  - USB connector ESD
  - CAN connector ESD

Optional robustness features (pick at least one):
- hardware current sense on input (shunt + amplifier footprint)
- output high-side switch for `+5V_OUT` pin
- thermal sensing (NTC + divider into ADC)

---

### 3.10 Observability & debugging
Board must include:
- **at least 4 LEDs**
  - `PWR_5V`
  - `PWR_3V3`
  - user LED (GPIO)
  - fault or status LED (GPIO)
- at least **15 test points**, including:
  - VIN_BAT, USB_5V, +5V, +3V3, GND
  - NRST
  - UART TX/RX
  - CAN TX/RX (logic side)
  - QSPI CLK and NCS (or equivalent)

---

## 4. PCB / layout requirements

### 4.1 Stackup
- **2-layer board**
- FR4
- 1 oz copper

### 4.2 Board size & mechanical
- Must be Nucleo-ish in size:
  - target outline: **70 mm x 100 mm** (approx)
  - maximum: **100 mm x 100 mm**
- 4x mounting holes:
  - M3 preferred
  - keep copper clearance

Board markings:
- board name: `F446ZET ENHANCED NUCLEO`
- revision: `REV A`
- date
- designer name/initials

### 4.3 Placement guidance (must follow)
- keep power input + regulators clustered
- switching regulator high-current loop must be compact
- keep QSPI flash close to MCU QSPI pins
- keep CAN transceiver close to CAN connector
- keep USB ESD close to connector

### 4.4 Design rules (must define in Altium)
Must define:
- net classes:
  - high current (VIN / +5V)
  - digital signals
  - QSPI signals
- trace width rules:
  - VIN and +5V wider than signals
- clearance rules:
  - set to match your fabrication capabilities (eg 6/6 mil)
- if QSPI used:
  - define length matching constraints (loose is fine)

---

## 5. Library & part selection rules

### 5.1 Allowed sources
- datasheets are primary source
- footprints must be verified
- avoid using unknown community footprints without validation

### 5.2 Mandatory checklist for each custom footprint
For each custom/added footprint you must document:
- source (datasheet reference)
- dimensions verified (Y/N)
- pin-1 / polarity marking confirmed (Y/N)
- courtyard rules defined (Y/N)

---

## 6. Deliverables

### 6.1 Design files (Altium)
- `.PrjPcb`
- schematic sheets (cleanly separated):
  - Power
  - MCU core + clocks + reset
  - CAN
  - External Flash
  - USB-UART + connectors
  - Expansion headers
- PCB layout

### 6.2 Manufacturing outputs
- gerbers
- drill files
- board outline included
- correct solder mask and paste layers

### 6.3 Assembly outputs
- BOM including:
  - Manufacturer Part Number (MPN)
  - quantity
  - designators
- pick-and-place / centroid
- assembly drawing PDF (recommended)

---

## 7. Testing & review

### 7.1 Design review checklist (minimum)
Before marking complete:
- ERC/DRC clean (or justified)
- no unrouted nets
- all connectors labelled and oriented correctly
- pin-1 markings visible
- test points present for key nets
- termination select jumper clearly labelled
- boot config documented (BOOT0 defaults)

### 7.2 “Student misuse” scenario
At minimum state what happens for:
- USB plugged in + battery connected at same time
- battery reverse polarity
- CANH/CANL shorted to GND
- CANH/CANL shorted together
- ESD strike on USB connector
- ESD strike on CAN connector
