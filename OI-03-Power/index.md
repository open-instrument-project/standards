---
id: OI-03
title: Open Instrument Power
version: 0.1.0
status: Draft
draft: 1
date: 2026-08-16
license: Apache-2.0
---

# OI-03 — Open Instrument Power

`SPDX-License-Identifier: Apache-2.0`

**Version:** 0.1.0<br>
**Status:** Draft 1<br>
**Date:** 2026-08-16<br>
**Project:** [Open Instrument Project](https://openinstrument.dev)

> **Early draft — discussion encouraged:** This standard is still being shaped. Please start proposed changes, interoperability feedback, and open design questions in the community [Ideas & Proposals](https://github.com/open-instrument-project/community/discussions/categories/ideas-proposals) discussions. Once work becomes concrete and actionable, move it to a [standards Issue](https://github.com/open-instrument-project/standards/issues) or pull request. See the [project-wide contribution policy](https://github.com/open-instrument-project/community/blob/main/CONTRIBUTING.md).

> **Service power for the instrument, not a universal power path for the DUT.**

## 1. Purpose

Open Instrument Power (**OI Power**) defines a common low-voltage DC **service/housekeeping power** interface for compatible instruments, especially instruments built in the OI Stack form factor.

Adoption of OI Power is optional and independent. A project may claim OI-03 without claiming OI-01, OI-02, OI-04, or OI-05; the requirements in this document apply only to projects claiming OI Power compatibility.

OI Power exists so a collection of instruments can share one service-power source instead of requiring a separate wall adapter for every device.

Typical loads include:

- MCU and digital control electronics;
- displays and indicators;
- OI Link transceivers;
- measurement/control electronics;
- nonvolatile storage;
- relays and low-energy actuators;
- local 5 V / 3.3 V conversion;
- cooling fans and thermal-control electronics.

OI Power is **not** intended to be a universal DUT-energy bus.

An instrument that sources, sinks, or otherwise handles substantial DUT power may use a separate instrument-specific energy connector while continuing to use OI Power for its control/service domain.

## 2. Service-alive principle

A central Draft 1 principle is:

> When OI Power is present, a compatible stack instrument should be alive as an instrument even if its instrument-specific functional-energy source is absent.

For a high-power instrument this may mean:

```text
OI Power present:
    MCU alive
    display/status alive
    OI Link alive
    configuration available
    faults/status queryable

functional-energy input absent:
    high-power source/load path unavailable
```

OI Power does not imply that every functional output of the instrument can operate.

A device SHALL report unavailable functionality explicitly through OI Protocol rather than silently pretending that full capability exists.

## 3. Nominal bus voltage

Draft 1 nominal bus voltage:

```text
12 V DC nominal
```

The previous 24 V working assumption is superseded.

The shift to 12 V reflects the clarified role of OI Power as a service/system-power rail rather than a high-power instrument backplane.

Exact permitted voltage tolerance is **TBD**.

## 4. Draft bus-power target

Current Draft 1 platform target:

```text
nominal voltage:       12 V
maximum chain current: approximately 5 A
nominal maximum bus:   approximately 60 W
```

The 5 A / 60 W values remain provisional until the connector, cabling, thermal behavior, inrush rules, and protection scheme are mechanically/electrically validated.

An OI Power source MAY provide less than the platform maximum.

The available budget SHALL be discoverable through the system/protocol when active power management is implemented.

## 5. Daisy-chain topology

An instrument claiming both OI Power and OI Stack is expected to provide an OI Power input and pass-through/output in the standardized OI Stack locations.

Conceptually:

```text
Source / OI Base
      |
      +-- PWR IN  Slice 1  PWR OUT --+
                                      |
                                      +-- PWR IN  Slice 2  PWR OUT --+
                                                                          |
                                                                          ...
```

The upstream connector, conductors, PCB copper, and pass-through path of a Slice may carry the **total downstream stack current**, not merely the Slice's own consumption.

That distinction SHALL be considered in every compatible design.

## 6. Power budgeting

Each device should declare an exact maximum OI Power draw rather than forcing software to infer it from product identity.

Conceptually:

```yaml
oi_power:
  nominal_voltage: 12 V
  maximum_declared_draw: 8 W
  current_draw: 4.2 W
  role: service
```

A future coarse power-class system MAY be added for convenience, but Draft 1 does not freeze class thresholds.

The system should support:

```text
available service-power budget
- declared/allocated instrument requirements
= remaining service-power budget
```

An OI Base or other source may monitor total bus voltage/current/power.

Per-device current monitoring is optional unless a later revision requires it.

## 7. Instrument-specific functional energy

OI Power deliberately does not standardize all high-energy paths.

Examples may include:

- dedicated DC input to a programmable supply;
- DUT input terminals of an electronic load;
- high-current battery connections;
- instrument-specific isolated source input;
- other application-specific connectors.

Such energy paths are outside OI-03 unless another standard explicitly defines them.

The relationship between OI Power and an instrument-specific energy path SHALL be documented.

OI Power SHALL NOT be back-driven from an instrument-specific source.

## 8. Local conversion

Each instrument is responsible for converting the 12 V service bus into its internal rails.

Typical architecture:

```text
12 V OI Power
     |
input protection
     |
local conversion
     |
 +---+----+----------+
 |        |          |
3V3      5V      other service rails
```

OI Power does not guarantee native 5 V, 3.3 V, analogue rails, or isolated rails.

## 9. Connector direction

The current mechanical direction is a commodity coaxial DC barrel family:

```text
approximately 5.5 mm outer diameter
approximately 2.5 mm centre pin
centre-positive
```

This is **not yet frozen as a normative connector specification**.

The final connector decision must verify:

- at least the intended full chain current;
- safe repeated bench use;
- keyed/polarized behavior;
- availability from multiple mainstream sources;
- PCB/panel mounting;
- practical short daisy-chain cables;
- clear distinction from instrument-specific higher-energy connectors.

No particular manufacturer part number is normative in Draft 1.

## 10. Polarity

The intended polarity for the candidate coaxial connector is:

```text
centre: +12 V
outer:  return
```

Final mechanical drawings and connector tolerances remain TBD.

## 11. Protection

Each instrument SHALL protect its own OI Power input appropriately.

The final normative requirements are not frozen, but design coverage is expected for:

- over-current/fusing;
- reverse polarity;
- over-voltage;
- input transients;
- inrush;
- local short circuits;
- pass-through faults;
- thermal overload;
- backfeed from USB or instrument-specific supplies.

A local instrument failure should not unnecessarily disable the rest of the chain.

Protection coordination with OI Base or another service-power source remains open.

## 12. Pass-through behavior

The final power pass-through architecture is not frozen.

The design goal is that an instrument's local off/fault state should not unnecessarily remove service power from downstream instruments.

Possible implementation styles include:

- passive protected pass-through;
- fused pass-through;
- electronically managed pass-through.

Any architecture must still respect the total downstream current requirement.

## 13. Grounding and isolation

OI Power naturally creates a shared **service/control power domain** unless isolation is deliberately introduced.

That shared return SHALL NOT silently become the DUT or precision-measurement reference where doing so would violate the instrument's stated isolation.

Conceptually:

```text
OI SERVICE DOMAIN
    12 V service
    MCU / UI / comms
    OI Link
         |
    isolation as required
         |
INSTRUMENT / DUT DOMAIN
    precision analogue
    source/load path
    measurement reference
```

Every instrument claiming OI Power SHALL document the relationship between:

- OI Power return;
- OI Link/control ground;
- OI Standalone/USB ground;
- chassis/shield;
- DUT/measurement ground;
- instrument-specific power returns.

Not every device requires galvanic isolation, but the grounding behavior must be explicit.

## 14. Relationship to OI Standalone

OI Power is **not required for firmware recovery**.

If a device also claims **OI-04 — Open Instrument Standalone**, its recovery path must work from USB-C alone.

A device may also support normal standalone operation from USB-C.

When both OI Power and USB power are present, the instrument is responsible for a safe power-path implementation with no prohibited backfeed.

## 15. Relationship to OI Stack

OI-05 defines the physical placement of OI Power connectors and requires OI-03 as part of an OI Stack claim. OI Power itself may still be implemented without OI Stack.

A Stack-compatible high-power instrument may therefore be physically present and fully discoverable while its separate functional-energy input is absent.

This service-alive behavior is intentional.

## 16. Draft 1 compliance intent

A device claiming Draft 1 OI Power compatibility should provide:

- 12 V nominal service-power operation;
- declared maximum OI Power consumption;
- correct polarity;
- adequate input protection;
- no backfeed into the OI Power bus;
- documented grounding/isolation behavior;
- safe coexistence with USB/instrument-specific power;
- pass-through behavior if the device claims OI Stack compatibility.

A source claiming Draft 1 OI Power source compatibility should:

- provide a protected 12 V service supply;
- declare its available power/current budget;
- not exceed the final voltage limits once frozen.

## 17. Open decisions

Still open:

- exact allowable 12 V tolerance;
- final maximum chain current;
- final maximum service-bus power;
- exact connector dimensions/gender;
- cable construction/current rating;
- hot-plug requirement;
- inrush limits;
- passive vs electronic pass-through;
- source/load protection coordination;
- whether coarse power classes are useful;
- minimum/reference OI Base capability;
- mandatory bus telemetry.

## 18. Draft history

### Draft 1 — 2026-08-16

First consolidated OI-03 draft based on a 12 V service-power architecture and explicit separation between service power and instrument/DUT energy.
