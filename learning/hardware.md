# Hardware design and wiring — revision H1

## Scope and purchases

Use [shopping-list.md](shopping-list.md) for exact links. Cart A runs the first two modules, Cart B supports physical inputs, Cart C supplies the motor circuit, and mechanics are added before M07. A fixed prototype pinout avoids teaching conflicting examples. Confirm board markings before using it.

Reference controller: NUCLEO-F401RE (MB1136 family) with onboard ST-LINK/V2-1. Reference motor: four-wire JK42HW34-0334, 200 full steps/revolution, rated 0.33 A/phase. Reference driver: A4988 carrier, 3.3 V logic, separate 12 V motor supply. Use full-step mode initially. No heavy payload, vertical axis or unattended movement in the core course.

## Pin ownership

The MCU signal name is authoritative. Locate the corresponding header using **the NUCLEO-F401RE table** in UM1724, not a different Nucleo model's table. Verify solder bridge variants for A4/A5.

| Function | MCU signal | Convenient header | Configuration / owner |
|---|---|---|---|
| Status LED LD2 | PA5 | D13, already wired onboard | GPIO output |
| STEP | PA8 | D7 | GPIO in M04, TIM1_CH1 AF1 from M05 |
| DIR | PB10 | D6 | GPIO output, defined low at boot |
| Driver ENABLE, active low | PC7 | D9 | GPIO output; external 10 kΩ pull-up to 3.3 V |
| User STOP button | PB0 | A3 | Input pull-up; NO contact to GND, pressed=low |
| HOME limit | PC0 | A5 normally; verify bridges | 10 kΩ pull-up; COM to GND, NC to input; actuated/open wire=high |
| FAR limit | PC1 | A4 normally; verify bridges | Same NC arrangement |
| Pulse observation loopback | PA0 | A0 | TIM2_CH1 AF1 external-clock counter; jumper STEP to PA0 |
| Timing trace (optional) | PB5 | D4 | GPIO output |
| Host serial TX/RX | PA2 / PA3 | D1 / D0, internal ST-LINK route | USART2, 115200 8N1; do not add another USB-UART adapter |
| SWD | PA13 / PA14 | Onboard debugger | Reserved; retain Serial Wire |

TIM1 owns STEP. TIM2 counts actual electrical rising edges on the loopback, introduced in M05. TIM5 is reserved for the HAL timebase if FreeRTOS uses SysTick. Do not allocate the same timer for unrelated tasks. PB0 and PC0 share EXTI line 0: they cannot both be independently routed to EXTI0. Poll user STOP; if HOME uses EXTI0, configure the source as port C. FAR can use EXTI1.

## Power topology

- Mac USB powers the Nucleo and its 3.3 V logic output.
- Enclosed 12 V adapter powers **only** A4988 VMOT and its motor power return.
- Join Nucleo ground, A4988 logic ground and motor supply negative at a deliberate common connection. Route motor return directly to the supply return rather than through a chain of signal jumpers.
- Supply A4988 VDD from Nucleo 3.3 V. Never apply the 12 V adapter to a Nucleo GPIO, 3V3 or 5V pin. Do not use VIN in this course.
- Put a 100 µF, 35 V electrolytic close across VMOT/GND, observing polarity; add 100 nF logic decoupling if not present on the carrier. The striped capacitor lead is negative.
- Use soldered/terminal connections for the motor-current path. The breadboard and thin Dupont leads are for signals. Cart C includes the materials for a small driver carrier assembly.

Keep the board and driver on a nonconductive base. Establish a readily reachable way to disconnect the low-voltage motor supply. This is independent of the software STOP button; neither is a certified emergency-stop circuit.

## Driver connections

| Driver pin | Connection |
|---|---|
| VDD, logic GND | 3.3 V and common GND |
| VMOT, motor GND | 12 V and supply return, bulk capacitor locally |
| STEP | PA8 plus external 10 kΩ pull-down |
| DIR | PB10 plus external 10 kΩ pull-down |
| ENABLE | PC7 plus external 10 kΩ pull-up (disabled during MCU reset) |
| RESET and SLEEP | Tie together and to VDD; allow at least 1 ms after waking before steps |
| MS1, MS2, MS3 | GND for full steps; change only with motor supply off and document it |
| 1A/1B | Both ends of one motor coil, identified by continuity/resistance |
| 2A/2B | Both ends of the other coil |

Confirm each pin from the carrier's labels; orientation and potentiometer location differ among clones. Do not use wire colours as the only coil identification. The standby current setting remains relevant even when the motor is not rotating.

## Current limit before motor operation

Read the two sense-resistor markings on the actual carrier. Do not copy a VREF from a video. For the A4988, `I_limit = VREF / (8 × R_sense)`. At a conservative starting limit of 0.25 A, R050 requires 0.100 V and R100 requires 0.200 V. These are examples, not identification of the shipped carrier. Keep the limit at or below the motor's 0.33 A rating; do not compensate for missed steps by turning it up blindly.

Set VREF with the motor disconnected, the reference/logic supply configured as required by that carrier, the driver disabled and a DC-voltage meter on the correct terminals. Measure between GND and the verified VREF point. Secure the black lead; avoid slipping onto neighbouring pins. If the sense resistor cannot be identified, postpone power-up and obtain the board schematic or a documented replacement. Do not guess.

After adjustment, disconnect supplies, wait for the capacitor to discharge and confirm near-zero VMOT before attaching motor leads. Never connect/disconnect a coil or change wiring while energized. A DC supply-current reading is not the phase current. In full-step operation, the phase current may be about 0.7 of the configured peak limit; this is not permission to exceed the motor's rating.

## First-power sequence

Before power: inspect joints and capacitor polarity; check for shorts; verify coil pairs and supply polarity; load firmware that holds ENABLE high, STEP low and DIR defined. Logic power first. Confirm disabled output and current-limit setting. Apply motor power with the motor firmly mounted, then issue a bounded low-speed move. On error, disable the driver and disconnect motor power before investigating. Unexpected heat, erratic movement or repeated reset requires inspection, not additional current.

For breakpoint sessions, disconnect motor power. A halted CPU does not inherently stop a peripheral timer. Later lessons teach timer debug-freeze settings, but physical isolation remains the default debug procedure.

## Mechanics contract, selected in M07-L01

Use one small horizontal carriage, roughly 100–300 mm useful travel, driven by the existing NEMA17 motor through a matched belt/pulley or lead-screw mechanism. Record: motor mount pattern, 5 mm shaft coupling, screw **lead** per revolution (not just thread pitch), carriage travel, end-stop brackets, base dimensions and fastener lengths. A rail alone is not a complete drive. Prefer an assembled mechanism or matched kit over unrelated bargain components.

Install two NC switches before automated homing. They must trip before the physical hard stops with enough margin for the tested stopping behavior. Start with an empty carriage and low speed. Clamp the base; route wiring away from moving parts. Microstepping changes pulses/mm; changing it invalidates calibration and requires homing.

The course can develop and test some firmware with a shaft pointer or simulated switches while saving for a stage. Mark physical homing/repeatability criteria **Verification pending** until a real mechanism is measured. Do not mark the full project complete using simulated carriage motion.

## Source of truth

Electrical connections: ST UM1724 and STM32F401 datasheet. Timer/Flash behavior: RM0368. Driver behavior: Allegro A4988 datasheet; Pololu's carrier documentation explains the electrical principles, but its resistor value must not be assumed for the cheaper OEM carrier. See [sources.md](sources.md).
