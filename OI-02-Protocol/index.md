---
id: OI-02
title: Open Instrument Protocol
version: 0.1.0
status: Draft
draft: 1
date: 2026-08-16
license: Apache-2.0
---

# OI-02 — Open Instrument Protocol

`SPDX-License-Identifier: Apache-2.0`

**Version:** 0.1.0<br>
**Status:** Draft 1<br>
**Date:** 2026-08-16<br>
**Project:** [Open Instrument Project](https://openinstrument.dev)

> **Early draft — discussion encouraged:** This standard is still being shaped. Please start proposed changes, interoperability feedback, and open design questions in the community [Ideas & Proposals](https://github.com/open-instrument-project/community/discussions/categories/ideas-proposals) discussions. Once work becomes concrete and actionable, move it to a [standards Issue](https://github.com/open-instrument-project/standards/issues) or pull request. See the [project-wide contribution policy](https://github.com/open-instrument-project/community/blob/main/CONTRIBUTING.md).

> **Capabilities matter more than model numbers.**

## 1. Purpose

Open Instrument Protocol (**OI Protocol**) defines the common conceptual model used by instruments implementing the Open Instrument standards.

Adoption of OI Protocol is optional and independent. A project may claim OI-02 without claiming OI-01, OI-03, OI-04, or OI-05; OI-02 is also a required dependency of OI Link and OI Stack. The requirements in this document apply only to projects claiming OI Protocol compatibility.

It defines how a device:

- identifies itself;
- exposes resources and capabilities;
- accepts configuration and control requests;
- reports requested and actual state;
- reports measurements and telemetry;
- reports warnings and faults;
- describes available power/functionality;
- configures deterministic trigger behavior;
- participates in multi-instrument orchestration and composition.

OI Protocol is intended to be **transport-independent**.

When OI Protocol is combined with OI Standalone or OI Link, the same instrument semantics should be usable through either transport:

```text
Host -- OI Standalone / USB-C --> Instrument
```

and:

```text
Host -- gateway/coordinator -- OI Link --> Instrument
```

Applications should not require a different conceptual driver merely because the transport changed.

## 2. Design principles

OI Protocol should enable:

- discovery without model-specific host drivers;
- stable machine-readable identity;
- capability-based control;
- multiple instances of the same instrument class;
- dynamic capability reporting;
- generic host software;
- explicit requested-vs-actual state;
- deterministic trigger configuration;
- structured measurements and faults;
- composition of multiple physical instruments;
- future third-party implementations;
- backward-compatible extension where possible.

The protocol should describe **what an instrument can do**, not force all products into identical internal implementations.

## 3. Device identity

Every device SHALL expose machine-readable identity information sufficient for software and humans to distinguish it.

Draft 1 identity fields should include:

```text
device class
manufacturer / project
model
hardware revision
serial number or unique device identifier
firmware version
supported OI Protocol version(s)
implemented OI standards/capabilities
```

Illustrative representation:

```yaml
class: power_supply
manufacturer: Example Instruments
model: ThreeChannelSupply
serial: SUP-00142
hardware_revision: A
firmware: 0.4.1
protocol:
  major: 0
  minor: 1
```

The wire serialization is not frozen.

## 4. Resources and channels

A device may expose one or more addressable resources.

Examples:

```text
power supply
    channel 1
    channel 2
    channel 3

DAQ
    analogue input 1
    analogue input 2
    ...

digital I/O
    GPIO bank
    relay 1
    relay 2
```

Host software should be able to enumerate resources without product-specific knowledge.

Resource identifiers should remain stable for the duration of a device session and, where practical, across reboots.

Exact addressing rules remain TBD.

## 5. Capability model

A core rule is:

> Host software asks what the device/resource can do, then operates within the advertised capability set.

A power-supply channel might conceptually expose:

```yaml
class: power_supply_channel

voltage:
  minimum: 0.8 V
  hardware_maximum: 15 V

current:
  minimum: 0 A
  hardware_maximum: 5 A

power:
  hardware_maximum: 20 W

modes:
  - CV
  - CC

measurements:
  - voltage
  - current
  - power

trigger:
  input: true
  output_action: true
```

The exact schema and units encoding are not frozen.

## 6. Static capability vs current availability

OI Protocol SHALL distinguish between:

1. **hardware/design capability** — what the device can do under qualifying conditions; and
2. **currently available capability** — what it can do now given power source, temperature, interlocks, connected accessories, or other runtime constraints.

This distinction is necessary for devices whose usable envelope changes dynamically.

Illustrative representation:

```yaml
hardware:
  voltage_max: 15 V
  current_max: 5 A
  channel_power_max: 20 W
  aggregate_power_max: 60 W

available:
  voltage_max: 15 V
  aggregate_power_max: 48 W
  reason: input_power_limit

power_source:
  type: usb_pd
  contract_voltage: 20 V
  contract_current: 3 A
```

A host must not need to infer current availability from model number or raw supply voltage.

## 7. Requested state vs actual state

Control requests and actual electrical/mechanical state SHALL be distinguishable.

For example:

```yaml
output:
  enable_requested: true
  enabled_actual: false
  reason: aggregate_power_limit
```

A command to enable an output, close a relay, arm a trigger, or apply a setpoint may be:

- accepted;
- accepted with a documented clamp;
- rejected;
- accepted but not yet achieved.

The protocol should make these outcomes explicit.

A device SHALL NOT report a requested state as though it were already physically true when a fault, interlock, power limit, startup sequence, or hardware response prevents it.

## 8. Common operations

Common conceptual operations should exist where useful.

Examples:

```text
get_identity()
get_capabilities()
get_available_capabilities()
get_status()
get_faults()
clear_faults()
get_measurements()
arm()
disarm()
configure_trigger(...)
software_trigger()
```

Instrument-specific conceptual examples:

```text
Power supply:
    set_voltage(channel, 3.3 V)
    set_current_limit(channel, 1.0 A)
    output_enable(channel)

Electronic load:
    set_mode(CC)
    set_current(4.5 A)
    input_enable()

Multimeter:
    set_function(DCV)
    set_range(AUTO)
    get_reading()
```

The syntax shown is illustrative, not normative.

## 9. Measurements and units

A measurement representation should be able to express at least:

- quantity;
- numeric value;
- unit;
- resource/channel;
- validity/status;
- optional timestamp;
- optional range;
- optional resolution;
- optional uncertainty/calibration state.

A coherent snapshot mechanism is desirable where several values represent the same acquisition instant.

The canonical units system and wire encoding remain TBD.

## 10. Faults, warnings, and diagnostics

Devices SHOULD report structured machine-readable conditions rather than only text strings.

Examples include:

- over-voltage;
- over-current/protection trip;
- over-temperature;
- fan failure;
- calibration invalid;
- external functional power missing;
- OI Power unavailable;
- input power budget exceeded;
- measurement over-range;
- interlock open;
- trigger not armed;
- communication failure to an internal subsystem.

The model should distinguish at least:

- informational state;
- warning;
- recoverable fault;
- latched fault;
- condition requiring user intervention.

Whether a condition forces a resource off is device-specific but SHALL be queryable.

## 11. Power-awareness

Devices implementing OI Power SHOULD expose their service-power requirements and current power state.

A device that also requires instrument-specific functional energy SHALL report that distinction.

Illustrative model:

```yaml
oi_power:
  present: true
  maximum_declared_draw: 8 W
  role: service

functional_power:
  required_for_full_operation: true
  present: false

device_state:
  control_available: true
  full_function_available: false
```

This permits a stacked high-power instrument to remain alive, discoverable, and configurable from OI Power while reporting that its high-energy functional path is unavailable.

Exact field names are not frozen.

## 12. Trigger model

When OI-02 and OI-01 are claimed together, OI Protocol configures behavior associated with the hardware trigger defined by **OI-01**.

Typical sequence:

```text
1. Discover devices/resources.
2. Configure each resource.
3. Select one trigger source.
4. Configure trigger listeners/actions.
5. Arm participants.
6. Confirm readiness.
7. Generate hardware trigger.
8. Collect resulting status/measurements.
```

The protocol should support:

- trigger receiver enable/disable;
- edge/polarity configuration where supported;
- arm/disarm;
- trigger action selection;
- source capability discovery;
- selection of the single active trigger source;
- ready/armed reporting;
- trigger counters/timestamps where available.

Exact messages remain TBD.

## 13. Composition and virtual instruments

Composition is a first-class design goal.

Examples:

```text
2 x electronic load      -> larger virtual load
4 x DAQ                  -> more acquisition channels
power supply + load      -> battery cycler
DAQ + mux + I/O + source -> automated DUT tester
```

Physical devices remain independently discoverable.

A virtual instrument may be implemented by host software or a future coordinator.

OI Protocol should expose enough capability and constraint information for orchestration software to determine whether resources can be combined safely.

## 14. OI Standalone relationship

**OI-04 — Open Instrument Standalone** provides direct USB-C host access when it is also claimed.

For a project claiming OI-02 together with OI-01 and OI-04, the conceptual device/resource model exposed over OI Standalone SHALL map to the same underlying capabilities exposed over OI Link.

A device MAY additionally expose a human-friendly interface such as:

- USB CDC;
- SCPI-like text commands;
- vendor-specific diagnostics;

but these should map to the same underlying instrument state rather than create a second contradictory API.

## 15. Firmware and lifecycle information

The protocol should expose:

- firmware version;
- hardware revision;
- boot/application state;
- compatibility information;
- recovery/update state where meaningful.

Guaranteed firmware recovery, when OI-04 is claimed, is defined by **OI-04** and SHALL NOT depend on OI Link.

Firmware update over OI Link may be standardized later but is not required by Draft 1.

## 16. Candidate wire/application foundations

The final application-layer implementation is not frozen.

Current candidates include:

- **Cyphal over CAN-FD** — currently the leading candidate because it already addresses distributed-node concepts such as discovery/addressing, registers/configuration, diagnostics, transfer semantics, and time-related infrastructure;
- **CANopen FD** — retained as an established alternative/reference point;
- a smaller OI-specific layer if existing systems prove unnecessarily complex.

The Open Instrument Project should reuse proven mechanisms where doing so reduces unnecessary invention, while preserving the instrument-focused capability model defined here.

## 17. Compatibility and extension

Protocol implementations require explicit version/compatibility information.

The intended evolution rules are:

- unknown optional capabilities should be safely ignorable;
- adding an optional capability should not break an older host;
- required behavior should be discoverable rather than assumed from model name;
- incompatible mandatory semantic changes require a new major protocol revision.

Detailed negotiation rules remain TBD.

## 18. Security and trust

Authentication, authorization, secure firmware update, and trust-policy requirements are not frozen in Draft 1.

The eventual design should avoid making basic local ownership/serviceability dependent on an unavailable cloud service.

In a project that also claims OI-04, security features must not defeat the guaranteed physical recovery principle without an explicitly documented ownership/recovery mechanism.

## 19. Draft 1 compliance intent

A Draft 1 implementation of OI Protocol is expected to expose:

- identity;
- resource/channel enumeration;
- static capabilities;
- current/dynamic availability;
- requested and actual state where they may differ;
- measurements/status;
- structured faults;
- trigger capabilities/configuration where implemented;
- service/functional power state where relevant;
- explicit protocol version information.

The wire format is not yet a Draft 1 interoperability guarantee.

## 20. Open decisions

Still open:

- Cyphal vs CANopen FD vs OI-specific wire layer;
- node addressing/discovery;
- serialization;
- canonical unit encoding;
- resource identifier format;
- measurement timestamp/time-sync model;
- fault schema;
- capability schema;
- request/response/error framing;
- virtual-instrument orchestration semantics;
- security/authentication model;
- firmware update over OI Link;
- mapping onto OI Standalone USB transport.

## 21. Draft history

### Draft 1 — 2026-08-16

First consolidated OI-02 draft, including explicit dynamic capability and requested-vs-actual-state semantics.
