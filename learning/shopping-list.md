# Shopping list — STM32 motion-controller course

Prepared **4 October 2026**. Prices below are displayed Polish retail prices including VAT, **excluding delivery**, checked while preparing the plan. Stock and prices can change before checkout. Quantities are purchase quantities, so a row marked “1 pack” already includes the pack's stated number of pieces.

**Buy Cart A to begin: 118.70 zł**, or **104.80 zł if you already have a suitable USB-C hub/adapter**. That is everything required for Modules 1–2. All Cart A items come from **Botland**, a Polish shop. No soldering, motor, laboratory supply or separate debugger is required to start.

For fewer orders, buy **A+B: 218.90 zł**, plus a small battery allowance if needed. This covers Modules 1–3. My budget recommendation is A first: prove flashing and debugging on your macOS 27 Mac before spending on the motor circuit. The course includes a compatibility gate because the current ST support listing does not establish macOS 27 support.

This shopping list is included in the ZIP at `learning/shopping-list.md` and also supplied as a separate file. All the course's instructions belong under `learning/`; future firmware/host-code locations are selected during lessons.

## Cart A — required before the first lesson

| Item / direct Polish shop link | Buy | Price | Why / compatibility |
|---|---:|---:|---|
| [STM32 NUCLEO-F401RE, Botland DIS-02217](https://botland.com.pl/stm32-nucleo/2217-stm32-nucleo-f401re-stm32f401re-arm-cortex-m4-5904422330613.html) | 1 | 99.90 zł | Course reference board: Cortex-M4, 512 KiB Flash, 96 KiB RAM, onboard ST-LINK and USB serial. Choose this exact board, not a bare F401 chip. |
| [USB-A to mini-USB-B data cable, 1.8 m, Lanberg](https://botland.com.pl/przewody-miniusb-20/15795-przewod-miniusb-b-a-20-lanberg-18m-czarny-5901969413595.html) | 1 | 4.90 zł | Plugs into the Nucleo's ST-LINK connector. **Mini-USB**, not micro-USB. |
| [USB-C male to USB-A female OTG adapter](https://botland.com.pl/przewody-usb-c/14216-adapter-usb-a-usb-c-otg-5901890041447.html) | 1, unless owned | 13.90 zł | Connects that cable to the Mac's USB-C port. Existing data-capable hub can replace it. |
| **Cart A total** | | **118.70 zł** | All three pages showed available stock when checked. |

A direct **data-capable USB-C to mini-USB-B** cable is also acceptable if already owned; it replaces both cable and adapter. Do not buy an extra ST-LINK, USB-UART converter, Arduino board or generic sensor kit for this course.

Software is a **0 zł purchase allowance**: ST tools, host compiler, Swift and Git have usable no-cost paths. Installation and exact working versions are established in M01. A paid IDE subscription, Windows computer or Raspberry Pi is not part of the shopping list.

## Cart B — required before M03, useful to order with A

| Item / link | Buy | Price | Purpose |
|---|---:|---:|---|
| [justPi 830-hole breadboard](https://botland.com.pl/plytki-stykowe/19943-plytka-stykowa-justpi-830-otworow-5904422328610.html) | 1 | 7.90 zł | Low-current signal/button prototyping. Motor supply paths will be soldered/terminal-connected. |
| [justPi 120 jumper leads: M–M, F–F, M–F](https://botland.com.pl/przewody-polaczeniowe/19946-zestaw-przewodow-polaczeniowych-justpi-20cm-3x40szt-m-m-z-z-m-z-120szt-5904422328702.html) | 1 set | 17.90 zł | Nucleo, breadboard, driver logic and timer loopback. |
| [Tact switch 6×6 mm / 4.3 mm THT, 5 pieces](https://botland.com.pl/tact-switch/377-tact-switch-6x6mm-43mm-tht-5szt-5904422307622.html) | 1 pack | 1.90 zł | One physical software STOP button, with spares. |
| [10 kΩ, ¼ W THT resistors, 30 pieces](https://botland.com.pl/rezystory-przewlekane/20150-rezystor-justpi-tht-cf-weglowy-14w-10k-30szt-5904422329280.html) | 1 pack | 2.60 zł | Input pull-ups and the driver's defined reset defaults. This pack also covers later modules. |
| [UNI-T UT33A+ multimeter](https://botland.com.pl/mierniki-uniwersalne/16717-miernik-uniwersalny-uni-t-ut33a-5901890044431.html) | 1, unless borrowed/owned | 69.90 zł | Voltage/resistance checks and driver VREF. Use resistance readings to check continuity if no audible mode is available. |
| **Cart B subtotal** | | **100.20 zł** | Listed products showed available stock when checked. |

Before checkout, confirm the meter includes probes and its required batteries; do not assume batteries are in the box. Budget **5–10 zł** for a pair of AAA batteries if needed, using [AAA battery option at Botland](https://botland.com.pl/baterie/9341-bateria-aaa-r3-lr03-alkaliczna-everactive-pro-4szt-5902020523369.html) or batteries already owned. The meter is reusable workshop equipment, not a disposable project part. Subtract 69.90 zł if you can borrow an equivalent meter with suitable low DC-voltage resolution.

## Cart C — before M04, first motor operation

These are additions to A+B, not a second set of those items. Buy after the Mac/debug gate passes. The cheap A4988 carrier requires inspection of its actual sense resistors and may require header soldering. The course teaches current limiting before movement.

| Item / link | Buy | Price | Purpose / fit |
|---|---:|---:|---|
| [JK42HW34-0334 stepper, 12 V / 0.33 A / 0.22 Nm](https://botland.com.pl/silniki-krokowe/3616-silnik-krokowy-jk42hw34-0334-200-krokowobr-12v033a022nm-5904422332044.html) | 1 | 64.90 zł | Four-wire NEMA17, 200 full steps/rev, 5 mm shaft; modest training-axis loads. |
| [A4988 red StepStick carrier](https://botland.com.pl/stepstick-sterowniki-do-silnikow-krokowych/2660-sterownik-silnika-krokowego-a4988-reprap-czerwony-5904422359201.html) | 1 | 17.90 zł | 3.3 V logic from the Nucleo and separate motor power. Do not assume a clone's sense-resistor value. |
| [Enclosed justPi 12 V / 2.5 A adapter, 5.5/2.1 mm DC](https://botland.com.pl/zasilacze-dogniazdkowe/22694-zasilacz-justpi-12v25a-do-arduino-wtyk-dc-5521mm-5904422383787.html) | 1 | 34.90 zł | Motor supply; confirm centre-positive polarity. Nucleo stays USB-powered. |
| [5.5/2.1 mm female DC socket with terminal clamps](https://botland.com.pl/szybkozlacza/7466-gniazdo-dc-55x21mm-z-szybkozlaczem-i-przyciskami-5904422310486.html) | 1 | 3.90 zł | Connect adapter to short local power wires without cutting its cable. |
| [100 µF / 35 V electrolytic capacitors, 10 pieces](https://botland.com.pl/kondensatory-elektrolityczne-tht/898-kondensator-elektrolityczny-100uf-35v-6x12mm-105c-tht-10szt-5903351248235.html) | 1 pack | 1.50 zł | One close to VMOT/GND; observe polarity. |
| [100 nF / 50 V ceramic capacitors, 10 pieces](https://botland.com.pl/kondensatory-ceramiczne-tht/210-kondensator-ceramiczny-100nf50v-tht-10szt-5903351248198.html) | 1 pack | 0.99 zł | Local logic decoupling if needed; useful spare parts. |
| [50×70 mm double-sided prototyping PCB, 2.54 mm grid](https://botland.com.pl/plytki-uniwersalne/2745-plytka-drukowana-uniwersalna-dwustronna-50x70mm-5904422303303.html) | 1 | 4.50 zł | Small soldered driver/power assembly. |
| [Female 1×40 header, 2.54 mm, 10 strips](https://botland.com.pl/gniazda-szpilkowe-goldpin/20029-listwa-goldpin-1x40-zenska-raster-254mm-10szt-justpi-5904422329174.html) | 1 pack | 9.90 zł | Cut two 8-position sockets for the driver; cutting can sacrifice a position. |
| [Male 1×40 header, 2.54 mm, 10 strips](https://botland.com.pl/gniazda-szpilkowe-goldpin/20031-wtyk-goldpin-1x40-prosty-raster-254mm-czarny-10szt-justpi-5904422329198.html) | 1 pack | 4.90 zł | Driver headers if unsoldered and signal connections on the carrier. |
| [2-pin screw terminals, **2.54 mm** pitch, 10 pieces](https://botland.com.pl/zlacza-ark/9443-zlacze-ark-raster-254mm-2-pin-10szt-5904422313340.html) | 1 pack | 11.50 zł | Motor coils and power on the matching prototyping grid. Do not substitute a different pitch without adapting the assembly. |
| [Black PVC insulating tape](https://botland.com.pl/tasmy-izolacyjne/18713-tasma-izolacyjna-rebel-013x19mm-x-182m-czarna-5901436785040.html) | 1, unless owned | 2.90 zł | Insulate exposed connections; provide separate mechanical strain relief. |
| **Cart C priced Botland subtotal** | | **157.79 zł** | Excludes the wire/base allowance and tools below. |

Also obtain **about 1–2 m of flexible copper power wire, approximately 0.5 mm²**, plus a nonconductive mounting base and cable restraint. Reuse suitable known wire/materials if owned. A small quantity is more economical than Botland's large industrial rolls. One Polish small-quantity option is [NEO-LED 2×0.50 mm² marked twin cable](https://neoled.com.pl/przewod_zasilajacy_czarny_0_5.html); allow **5–10 zł for wire**, excluding its separate delivery. Buying the same specification locally avoids a second parcel. This is the one suggested small electronics-supply exception to the Botland preference. Do not substitute fine AWG30 wrapping wire for the motor-power path.

Allow **10–30 zł** for wire plus base/restraint if none can be reused. Secure the motor to a suitable base/bracket; do not leave it loose on the desk. For M04 a simple secured bench fixture is sufficient; the final stage mount is chosen with its mechanism.

**Carts A+B+C: 376.69 zł in listed Botland products**, or approximately **392–417 zł including the small material/battery allowances**, before shipping and soldering tools. Reusing a hub, meter and consumables reduces this substantially.

## Tools for Cart C — borrow first to keep the budget down

No soldering equipment is needed for the first three modules. Before M04, you need access to a temperature-controlled iron with a suitable stand, solder with electronics flux, small cutters/stripper, a small screwdriver and a stable work holder. A friend/makerspace can help assemble the carrier while you learn inspection and measurement. Do not substitute twisted bare wires for soldering to save buying tools.

| Tool / link | Quantity | Cost basis |
|---|---:|---|
| [Yihua 936A-II station with stand, cleaner and holders](https://botland.com.pl/stacje-lutownicze/24596-stacja-lutownicza-yihua-936a-ii-z-chwytakami-120w-5908272000634.html) | 1 if buying | **199.00 zł checked**; an example complete basic station, not a claim of the lowest price anywhere. Borrowing is the cheapest route. |
| [Electronics solder: Botland selection](https://botland.com.pl/166-cyny-lutownicze) | Small pack | **12.50 zł observed** for Cynel LC99.3 SW26 10 g/1 mm; select that item or equivalent electronics flux-cored solder. This is a category link. |
| [Vorel side cutters, 160 mm](https://botland.com.pl/szczypce-boczne/1538-szczypce-tnace-boczne-160mm-vorel-4004740017-5906083400476.html) | 1 if needed | **9.90 zł listed**; use suitable finer tools if already owned. |
| [Botland hand-tool selection](https://botland.com.pl/1384-narzedzia) | Small flat screwdriver and wire stripper if needed | **20–50 zł allowance**, not a checked product quote. Choose a stripper covering the actual wire sizes. |

Allow approximately **245–300 zł** if buying this workshop setup from scratch, or **0 zł additional** if borrowing a complete setup including consumables. Have a heat-resistant work surface and ventilation/fume control; follow the solder/iron instructions. M04-L01 includes basic joint practice and inspection before assembling the powered circuit. No hot-air rework station or oscilloscope is required.

## Cart D — later, before M07

| Item | Quantity / cost | When to buy |
|---|---|---|
| [WK612 lever microswitch](https://botland.com.pl/czujniki-krancowe/929-wylacznik-czujnik-krancowy-mini-z-dzwignia-wk612-5904422309084.html) | **2 × 2.50 zł = 5.00 zł** | Two NC limits. Confirm COM/NC terminals on arrival. |
| Matched horizontal axis mechanism with rail/carriage, belt drive or lead screw, NEMA17 bracket, 5 mm shaft interface, base and fasteners | **300–700 zł planning allowance** | Select during preparation for M07-L01 after choosing available space/travel. This is not a quoted complete kit. |
| Ruler or caliper and end-stop mounting materials | Reuse, or **30–80 zł allowance** | Measure motion with stated resolution; no need to buy a precision gauge at the start. |

The **mechanics are intentionally not a buy-now shopping cart**. A reliable complete kit and exact mounting compatibility have not been selected; buying a rail, screw and coupler independently now could waste money. The M07 checklist specifies everything needed and the lesson must select compatible currently available parts before ordering. A NEMA23-only mechanism is not a direct fit for the chosen NEMA17 motor. An assembled axis can also include a motor, in which case avoid buying a redundant motor when revising the plan.

An approximate total through a real moving stage is **730–1,200 zł with borrowed soldering tools**, or **975–1,500 zł with the example new tool setup**, plus delivery. These rounded totals include the stated later-mechanics allowances, not guaranteed retail offers. You can complete M01–M06 without buying the mechanism, then decide how much to spend on it.

## Optional later purchases — do not add now

- Logic analyzer: useful if you want independent electrical timing evidence beyond loopback counts/on-target timestamps. Borrow first; verify current Apple Silicon/macOS support and 3.3 V inputs before selecting a model.
- Encoder, second motor/driver, touchscreen, CAN interface, custom PCB, enclosure and emergency-stop hardware for a larger machine: outside the core budget/course. Their design requirements depend on the extension.
- Spare A4988: convenient but not required. Correct assembly/current limiting is preferable to treating failures as expected consumption.

## Before checkout

- Choose **NUCLEO-F401RE** and the **mini-USB** data cable. A low-cost generic “STM32 board” is not an equivalent substitute for these lessons.
- Confirm pack quantities so you do not order 30 packs of resistors or ten motor drivers.
- Confirm current availability. Exact stock is not reserved by this document.
- Check what you already own or can borrow, especially hub, meter, tools and measurement ruler.
- Add delivery from the actual checkout. Several Botland pages showed delivery **from 8.99 zł**, but the final amount depends on basket and method; multiple orders add shipping.
- Keep receipts and exact product labels. Photograph the actual A4988 sense resistors before its current-limit lesson.

The shop pages linked in each row are the sources for prices and product identities. Electrical wiring and timing should follow the manufacturer documents in `learning/sources.md`, not a shop's generic description.
