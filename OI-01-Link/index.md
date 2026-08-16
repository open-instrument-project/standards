---
id: OI-01
title: Open Instrument Link
version: 0.1.0
status: Draft
draft: 1
date: 2026-08-16
license: Apache-2.0
---

# OI-01 — Open Instrument Link

`SPDX-License-Identifier: Apache-2.0`

**Version:** 0.1.0<br>
**Status:** Draft 1<br>
**Date:** 2026-08-16<br>
**Project:** [Open Instrument Project](https://openinstrument.dev)

> **Early draft — discussion encouraged:** This standard is still being shaped. Please start proposed changes, interoperability feedback, and open design questions in the community [Ideas & Proposals](https://github.com/open-instrument-project/community/discussions/categories/ideas-proposals) discussions. Once work becomes concrete and actionable, move it to a [standards Issue](https://github.com/open-instrument-project/standards/issues) or pull request. See the [project-wide contribution policy](https://github.com/open-instrument-project/community/blob/main/CONTRIBUTING.md).

> **CAN says what. The trigger says when.**

## 1. Purpose

Open Instrument Link (**OI Link**) is the wired instrument-to-instrument interface used by devices implementing the Open Instrument standards.

Adoption of OI Link is optional, but an OI-01 claim requires **OI-02 — Open Instrument Protocol** so electrically compatible devices also share instrument semantics. OI-03 Power, OI-04 Standalone, and OI-05 Stack remain optional. The requirements in this document apply only to projects claiming OI Link compatibility.

OI Link provides two complementary services over inexpensive, widely available twisted-pair cabling:

1. a robust multi-drop **CAN-FD communications bus** for discovery, configuration, commands, telemetry, diagnostics, and orchestration; and
2. a dedicated **differential hardware trigger bus** for events whose timing should not depend on USB latency, host operating systems, application scheduling, or CAN arbitration.

OI Link is deliberately not an instrument-power bus and does not carry DUT power.

## 2. Design principles

OI Link is intended to be:

- inexpensive to implement;
- based on commodity connectors and cables;
- straightforward to daisy-chain;
- usable without an electrical backplane;
- robust in laboratory and automated-test environments;
- electrically continuous through an unpowered intermediate device where practical;
- suitable for generic capability-based control rather than model-specific commands;
- separable from the instrument's DUT/measurement ground where required;
- extensible without consuming reserved conductors prematurely.

The interface deliberately separates **configuration/control** from **deterministic action**:

```text
OI Protocol over CAN-FD
    configure / identify / arm / report

OI hardware trigger
    execute prepared action now
```

## 3. Connector and cable

### 3.1 Connector

The Draft 1 connector is an **8P8C modular connector**, commonly referred to as RJ45.

A normal OI Link instrument implementation provides **two electrically equivalent ports**:

```text
OI LINK          OI LINK
[ 8P8C ]         [ 8P8C ]
```

The two ports are not logical input/output ports. They are two access points to the same bus segment and allow a linear chain without an external hub.

### 3.2 Cable

The baseline cable is ordinary **straight-through CAT5e or better twisted-pair cable**.

Normal operation SHALL NOT require a proprietary Open Instrument cable.

### 3.3 Passive continuation

Bus conductors SHOULD continue directly between the two connectors wherever practical.

A powered-down intermediate instrument SHOULD NOT intentionally interrupt CAN or trigger continuity between devices on either side of it.

The local transceivers tap the continuous bus rather than forwarding packets or trigger events in firmware.

## 4. Draft 1 pair allocation

| Twisted pair | 8P8C pins | Draft 1 function |
|---|---:|---|
| Pair 1 | 1 / 2 | CAN-FD |
| Pair 2 | 3 / 6 | Differential hardware trigger |
| Pair 3 | 4 / 5 | **RESERVED** |
| Pair 4 | 7 / 8 | **RESERVED** |

The two reserved pairs SHALL remain unallocated in Draft 1.

In particular:

- OI Link SHALL NOT use the reserved pairs for power;
- implementations SHALL NOT invent private functions on reserved pairs while claiming Draft 1 interoperability;
- future assignment requires a later OI-01 revision.

Potential future uses such as additional deterministic signals, clock distribution, or additional trigger resources have been discussed but are not assigned.

## 5. Not Ethernet

OI Link uses an Ethernet-style connector and Ethernet cabling, but **OI Link is not Ethernet**.

No Ethernet MAC/PHY interoperability is implied.

### 5.1 Misconnection objective

Because the connector is common, accidental connection to ordinary network equipment is foreseeable.

Draft 1 therefore sets the following design objective:

> An OI Link port SHOULD be designed so that accidental connection to common Ethernet equipment does not normally damage the OI device or the Ethernet equipment.

This does not mean OI Link is Ethernet-compatible, nor does Draft 1 guarantee survival against arbitrary passive PoE injectors, non-standard wiring, or other out-of-spec sources.

Protection and final misconnection test requirements remain to be frozen.

## 6. CAN-FD communications bus

CAN-FD is the Draft 1 communications physical/data-link technology.

It carries activities such as:

- device discovery and addressing;
- identity and capability exchange;
- configuration;
- instrument commands;
- measurements and telemetry;
- structured fault reporting;
- arming and trigger configuration;
- orchestration between instruments.

Application semantics are defined by **OI-02 — Open Instrument Protocol**.

### 6.1 Topology

The intended topology is a linear multi-drop bus:

```text
TERM                                                TERM
  |                                                   |
  +-- Instrument A -- Instrument B -- Instrument C --+
```

Long stubs SHOULD be avoided.

The physical ends of the CAN bus are terminated. Intermediate devices are not.

### 6.2 Bit rates

**TBD.**

Draft 1 does not freeze CAN arbitration or data bit rates.

The eventual baseline SHOULD prioritize:

- robustness;
- predictable operation across practical bench-length chains;
- tolerance of ordinary CAT5e/CAT6 cabling;
- easy implementation across mainstream microcontrollers and transceivers;

over headline throughput.

### 6.3 Ground reference and isolation

OI Link SHALL NOT create undocumented DUT or measurement-current paths.

Each instrument SHALL document the relationship between:

- OI Link transceiver reference;
- OI Power/control ground where present;
- OI Standalone/USB ground;
- chassis/shield;
- instrument/DUT ground.

Whether galvanic isolation at the CAN physical layer becomes mandatory for all implementations is **not frozen**.

Isolation remains an instrument-specific requirement where the measurement/source/load domain must remain independent of the shared control domain.

## 7. Hardware trigger bus

The hardware trigger is the defining deterministic-timing feature of OI Link.

The intended model is:

```text
CAN-FD:
    configure action
    configure source/listeners
    arm devices

TRIGGER EDGE:
    execute prepared action
```

Example:

```text
OI Protocol:
    Load: configure transient; arm
    DAQ:  configure capture; arm

OI Trigger:
                      NOW
                       |
             +---------+---------+
             |                   |
           Load                  DAQ
      changes current       starts capture
```

The trigger is a **shared electrical bus**. It is not forwarded hop-by-hop by firmware.

### 7.1 Trigger continuation

The trigger pair SHOULD continue directly between both OI Link ports:

```text
Port A                                      Port B

TRIG+  ------------------------------------ TRIG+
                     |
                     +-- local receiver
                     +-- optional local driver
TRIG-  ------------------------------------ TRIG-
```

### 7.2 Trigger ownership

Draft 1 deliberately uses a single-driver model:

- any instrument MAY implement trigger-source capability;
- exactly **one active trigger driver** SHALL be enabled on a given trigger bus at one time;
- all other participating devices are listeners;
- source ownership/configuration is established through OI Protocol before the experiment is armed.

More complex multi-source or wired-OR semantics are outside Draft 1.

### 7.3 Trigger electrical layer

**TBD.**

The trigger SHALL use a differential electrical layer suitable for multi-drop operation over ordinary twisted-pair cable.

M-LVDS/LVDS-like approaches remain candidates, but no transceiver family, signalling amplitude, common-mode range, or termination value is frozen.

The final electrical layer should provide:

- low and predictable propagation delay;
- low jitter;
- good noise immunity;
- multi-drop operation;
- defined contention behavior;
- practical cable length;
- straightforward protection.

## 8. Termination

OI Link contains two independent differential buses and therefore two separate termination problems.

For both CAN-FD and the final trigger PHY:

- the two physical ends of the chain require the appropriate termination;
- intermediate devices do not;
- termination state must be inspectable/configurable in a predictable way.

Manual termination is acceptable for early reference implementations.

Software-controlled or automatic end-of-chain termination MAY be standardized later.

Exact trigger termination remains dependent on the selected trigger PHY.

## 9. Relationship to other standards

- **OI-02 — Open Instrument Protocol:** required by OI Link; defines identity, capabilities, commands, trigger configuration, measurements, and faults carried over CAN-FD.
- **OI-03 — Open Instrument Power:** defines a separate 12 V service-power connection; OI Link itself carries no power.
- **OI-04 — Open Instrument Standalone:** provides direct USB-C host access and guaranteed firmware recovery.
- **OI-05 — Open Instrument Stack:** defines the physical placement of OI Link connectors on stack-compatible instruments.

OI Link does not replace OI Standalone.

## 10. Draft 1 compliance intent

A device claiming Draft 1 OI Link compatibility is expected to provide:

- two equivalent 8P8C OI Link ports for a normal daisy-chain instrument;
- the Draft 1 pair allocation;
- CAN-FD physical interface;
- passive CAN/trigger continuation wherever practical;
- a hardware trigger receiver;
- trigger-source hardware only if the device claims source capability;
- correct bus termination behavior;
- OI-02 compatibility and its shared identity, capability, command, trigger, measurement, and fault semantics over CAN-FD;
- documented grounding/isolation behavior;
- protection appropriate to an exposed laboratory connector;
- no use of reserved pairs.

## 11. Open decisions

The following remain intentionally open:

- CAN-FD arbitration/data rates;
- CAN transceiver requirements;
- mandatory vs optional CAN isolation;
- final trigger transceiver technology;
- trigger signalling levels/common-mode range;
- trigger termination;
- maximum chain length;
- maximum node count;
- automatic/electronic termination mechanism;
- exact accidental-Ethernet/PoE survivability requirement;
- future use of reserved pairs.

## 12. Draft history

### Draft 1 — 2026-08-16

First consolidated OI-01 draft using the two-digit standard ID, required OI Protocol semantics, and current Open Instrument Project architecture.
