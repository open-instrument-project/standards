---
id: OI-05
title: Open Instrument Stack
version: 0.1.0
status: Draft
draft: 1
date: 2026-08-16
license: Apache-2.0
---

# OI-05 — Open Instrument Stack

`SPDX-License-Identifier: Apache-2.0`

**Version:** 0.1.0<br>
**Status:** Draft 1<br>
**Date:** 2026-08-16<br>
**Project:** [Open Instrument Project](https://openinstrument.dev)

> **Early draft — discussion encouraged:** This standard is still being shaped. Please start proposed changes, interoperability feedback, and open design questions in the community [Ideas & Proposals](https://github.com/open-instrument-project/community/discussions/categories/ideas-proposals) discussions. Once work becomes concrete and actionable, move it to a [standards Issue](https://github.com/open-instrument-project/standards/issues) or pull request. See the [project-wide contribution policy](https://github.com/open-instrument-project/community/blob/main/CONTRIBUTING.md).

> **The stack is packaging. The instruments remain instruments.**

## 1. Purpose

Open Instrument Stack (**OI Stack**) is the common mechanical form factor for compact instruments implementing the Open Instrument standards.

Adoption of OI Stack is optional, but an OI-05 claim requires **OI-01 — Open Instrument Link**, **OI-02 — Open Instrument Protocol**, and **OI-03 — Open Instrument Power** so stacked instruments can be chained with shared communication, triggering, instrument semantics, and service power. OI-04 Standalone remains optional. The requirements in this document apply only to projects claiming OI Stack compatibility.

OI Stack standardizes:

- external XY footprint;
- height increments;
- mounting/alignment;
- front/rear datum planes;
- service-interface zones;
- connector positions;
- mechanical keep-outs;
- airflow conventions.

It deliberately does **not** introduce a proprietary high-density electrical backplane.

A stack-compatible instrument remains a recognizable, directly controllable instrument when removed from the stack.

## 2. Design principles

OI Stack should provide:

- a repeatable desktop footprint;
- simple physical stacking/alignment;
- standard height increments;
- predictable rear service connections;
- short commodity OI Link and OI Power jumpers;
- common airflow assumptions;
- room for instrument-specific front panels;
- room for instrument-specific power/IO where required;
- support for multi-U instruments;
- support for multiple identical slices.

It should avoid:

- proprietary card cages;
- mandatory central electronics;
- blind high-density backplane connectors;
- hidden proprietary interconnects;
- making an instrument useless outside its chassis.

## 3. No electrical backplane

OI Stack standardizes mechanics and connector placement, not a hidden system bus.

Communication and service power remain visible cable connections:

```text
             OI POWER        OI LINK
                 |              |
OI Base ---------+--------------+
                 |              |
Slice 1 ---------+--------------+
                 |              |
Slice 2 ---------+--------------+
                 |              |
Slice 3 ---------+--------------+
```

This is intentional:

- cables are easy to source;
- connections are inspectable;
- individual instruments remain serviceable;
- one proprietary backplane connector does not constrain the ecosystem.

## 4. Stack dependencies and service-alive behavior

An OI Stack claim SHALL include:

- **OI-01 — Open Instrument Link**, providing the chainable CAN-FD communications and hardware-trigger buses;
- **OI-02 — Open Instrument Protocol**, providing shared instrument semantics over OI Link; and
- **OI-03 — Open Instrument Power**, providing chainable service power.

OI Standalone is not required by OI Stack.

For every OI interface a stack-compatible project claims:

- the corresponding connector or control SHALL use the OI Stack location once that location is frozen;
- the implementation SHALL satisfy that other standard independently;
- unused interface locations MAY be left unpopulated, subject to the final mechanical keep-out rules.

Because every OI Stack instrument also claims OI Power, the OI Power service-alive principle applies. For a high-power slice, OI Power should normally support:

- MCU/control logic;
- OI Link discovery/control;
- status indication/display where present;
- diagnostics;
- configuration.

The slice MAY report its main functional path as unavailable until a separate energy source is connected.

The stack therefore remains inspectable and manageable without requiring a USB-C cable to every slice.

## 5. Common footprint

A common X/Y footprint SHALL be defined before OI-05 reaches Candidate status.

**Exact dimensions are not frozen in Draft 1.**

The eventual assets should define:

- PCB/reference outline;
- enclosure envelope;
- mounting-hole coordinates;
- front-panel datum;
- rear-panel datum;
- service connector coordinates;
- mechanical keep-outs;
- vent/airflow regions;
- reference STEP geometry.

## 6. Height units

Stack device heights use integer multiples of a common unit:

```text
1U
2U
3U
...
```

A value around **25 mm per U** has been used as an illustrative design target, but the actual unit height is not frozen.

A high-dissipation or mechanically large instrument MAY occupy multiple U.

## 7. Standard rear service zone

A stack-compatible instrument SHALL place required and optional OI interfaces in their corresponding defined rear-panel positions.

Current interface set:

```text
+--------------------------------------------------+
| OI LINK   OI LINK      OI STANDALONE RECOVERY   |
| [8P8C]    [8P8C]       [USB-C]          o       |
|                                                  |
| OI POWER IN            OI POWER OUT              |
| [TBD]                  [TBD]                     |
+--------------------------------------------------+
```

Exact coordinates/order are not frozen.

The two physical OI Link ports and OI Power input/pass-through are required. Required OI Protocol semantics run over OI Link and need no separate connector. OI Standalone and RECOVERY occupy their defined locations only when OI-04 is also claimed.

The intent is that short standard jumpers can be used repeatedly through a stack and that a user can identify the service connectors without learning every product layout.

## 8. Standard interfaces

### 8.1 OI Link

Two equivalent OI Link ports per **OI-01** are required and provide CAN-FD communications and hardware-trigger continuity. Communications over those ports SHALL use the shared instrument semantics required by **OI-02**.

### 8.2 OI Power

OI Power input/pass-through per **OI-03** is required and provides 12 V nominal service power.

The pass-through path may carry downstream stack current and must be designed accordingly.

### 8.3 OI Standalone

If OI-04 is claimed, one USB-C OI Standalone interface provides direct host access and firmware update.

### 8.4 RECOVERY

A project claiming OI-04 also locates its required physical RECOVERY control with OI Standalone.

Required OI Link/OI Power and optional OI Standalone physical services belong in the rear service zone so the front panel can remain instrument-specific.

## 9. Instrument-specific front panel

OI Stack does not prescribe the instrument's front-panel function.

Typical front-panel elements may include:

- displays;
- rotary encoders;
- channel buttons;
- 2 mm or 4 mm test terminals;
- BNCs;
- digital/relay terminals;
- DUT connectors;
- load/source connectors.

The standard should define only the shared mechanical envelope, datums, keep-outs, and safety/clearance constraints.

## 10. Instrument-specific power and I/O

A slice MAY expose additional rear or front connectors for capabilities outside the common service interfaces.

Examples include:

- dedicated higher-power DC input;
- battery/DUT power connector;
- isolated measurement input;
- RF connector;
- high-current load/source connection.

Such connectors:

- are not part of OI Power unless another standard explicitly defines them;
- SHALL NOT obstruct the standard service zone;
- SHALL respect the common mechanical envelope/keep-outs;
- should be physically difficult to confuse with OI Power where hazardous misuse is plausible.

## 11. OI Base

OI Base is a reference coordination/service-power device often shown at the bottom of example stacks.

Possible reference functions include:

- 12 V OI Power source/distribution;
- total service-bus power measurement/protection;
- USB-to-OI Link gateway;
- OI Link termination;
- hardware trigger generation.

OI Base is **not required by OI-05**.

A stack may be coordinated through another gateway. Instruments that also claim OI Standalone remain directly accessible through that interface.

The standard does not require Linux, Raspberry Pi, or any particular processor in OI Base.

## 12. Repeated and composable slices

The form factor explicitly supports multiple instances of the same instrument type.

Conceptual examples:

```text
2 x electronic load  -> more load capacity
4 x DAQ              -> more acquisition channels
2 x programmable PSU -> more independent rails
multiple I/O slices  -> more fixture automation
```

OI Protocol handles identity/capability discovery and composition semantics. OI-05 itself makes physical repetition predictable.

## 13. Independent use

Removing a slice from the stack SHALL NOT make ordinary ownership/service access impossible.

At minimum, the project's normal user interface, claimed service interfaces, and required instrument-specific power connections remain accessible outside a stack.

If OI Standalone is also claimed, direct USB-C communication and firmware recovery remain available. Low-power instruments implementing OI Standalone SHOULD operate usefully from USB-C alone where practical. High-power instruments MAY require an instrument-specific energy source for full operation.

## 14. Cooling and airflow

A common airflow convention should be part of the final mechanical standard.

It is especially important for:

- electronic loads;
- programmable supplies;
- instruments with active cooling;
- multi-U thermal assemblies.

The final standard should define:

- preferred intake/exhaust direction;
- vent keep-outs;
- clearance between slices;
- rules for side/rear/top ventilation;
- multi-U cooling behavior;
- how one slice avoids excessively preheating another.

**Airflow direction is not frozen in Draft 1.**

## 15. Structural design

The final mechanical standard should define reasonable limits/requirements for:

- mounting-hole pattern;
- screw/thread family;
- load-bearing structure;
- slice mass;
- total supported stack height;
- lateral stability;
- multi-U instrument support;
- heavy cooling assemblies.

Whether structural loads are carried by corner pillars, side frames, rails, or another scheme remains TBD.

## 16. Serviceability

A stack should be assemblable and serviceable with ordinary tools.

The design should avoid hidden fasteners or interconnects that require dismantling unrelated slices merely to access one device.

Cable replacement should not require replacement of a proprietary backplane.

Reference mechanical designs should favor parts that are commonly sourceable and reproducible.

## 17. Reference assets

Before OI-05 reaches Candidate status, the repository should provide reusable assets such as:

- KiCad board-outline/template;
- STEP envelope/reference model;
- front-panel DXF/template;
- rear service-zone DXF/template;
- connector keep-out models;
- reference 1U/2U/3U geometry;
- mounting-hardware BOM;
- example slice assembly.

A third-party designer should be able to start from the templates rather than reverse-engineering another product.

## 18. Compatibility marking

The project-wide OI Compatibility Badge reports each standard separately. OI Stack height is shown only when the OI-05 checkbox is selected, for example:

```text
OI COMPATIBILITY
[x] OI-01 Link
[x] OI-02 Protocol
[x] OI-03 Power
[ ] OI-04 Standalone
[x] OI-05 Stack (2U)
```

Branding/compatibility-mark rules are outside the Apache-2.0 licence for this document and will be defined separately.

## 19. Relationship to other standards

- **OI-01 — Open Instrument Link:** common rear communications/trigger ports.
- **OI-02 — Open Instrument Protocol:** identity, capabilities, orchestration, and composition.
- **OI-03 — Open Instrument Power:** 12 V service-power input/pass-through.
- **OI-04 — Open Instrument Standalone:** USB-C direct interface and RECOVERY behavior.

No OI standard is mandatory for every project. OI Link is a compound claim requiring OI Protocol. OI Stack is a compound claim requiring OI Link, OI Protocol, and OI Power; OI Standalone remains optional.

## 20. Draft 1 compliance intent

A Draft 1 stack-compatible instrument is expected to provide:

- eventual common X/Y envelope once frozen;
- integer-U height once frozen;
- OI-01 compatibility and two OI Link ports;
- OI-02 compatibility and shared identity, capability, state, telemetry, fault, trigger, and composition semantics;
- OI-03 compatibility and OI Power input/pass-through;
- OI Power service-alive behavior;
- standard rear service-zone placement for required and optional OI interfaces;
- instrument-specific front panel;
- documented cooling requirements;
- independent access to its normal user, service, and power interfaces outside the stack.

Mechanical dimensions are not yet sufficiently frozen for true physical interoperability.

## 21. Open decisions

Still open:

- X/Y footprint;
- exact 1U height;
- mounting-hole pattern;
- rear connector coordinates/order;
- OI Power connector geometry;
- auxiliary connector zone;
- airflow direction;
- vent keep-outs;
- side-wall/pillar/rail strategy;
- maximum slice mass;
- maximum stack height;
- multi-U structural rules;
- enclosure material/reference construction.

## 22. Draft history

### Draft 1 — 2026-08-16

First consolidated OI-05 draft, including the mechanical service zones and the required OI Link, OI Protocol, and OI Power dependencies.
