---
id: OI-04
title: Open Instrument Standalone
version: 0.1.0
status: Draft
draft: 1
date: 2026-08-16
license: Apache-2.0
---

# OI-04 — Open Instrument Standalone

`SPDX-License-Identifier: Apache-2.0`

**Version:** 0.1.0<br>
**Status:** Draft 1<br>
**Date:** 2026-08-16<br>
**Project:** [Open Instrument Project](https://openinstrument.dev)

> **Early draft — discussion encouraged:** This standard is still being shaped. Please start proposed changes, interoperability feedback, and open design questions in the community [Ideas & Proposals](https://github.com/open-instrument-project/community/discussions/categories/ideas-proposals) discussions. Once work becomes concrete and actionable, move it to a [standards Issue](https://github.com/open-instrument-project/standards/issues) or pull request. See the [project-wide contribution policy](https://github.com/open-instrument-project/community/blob/main/CONTRIBUTING.md).

> **One instrument, one host, one guaranteed recovery path.**

## 1. Purpose

Open Instrument Standalone (**OI Standalone**) is the direct USB-C host interface for an instrument implementing the Open Instrument standards.

Adoption of OI Standalone is optional and independent. A project may claim OI-04 without claiming OI-01, OI-02, OI-03, or OI-05; the requirements in this document apply only to projects claiming OI Standalone compatibility.

Its role is deliberately different from OI Link:

```text
OI Standalone = one instrument <-> one host
OI Link  = instrument <-> instruments
```

OI Standalone ensures that an instrument remains directly accessible, configurable, updateable, and recoverable without requiring an OI Base, an OI Link network, or an OI Stack.

## 2. Baseline services

A Draft 1 OI Standalone implementation SHALL provide:

- one USB-C receptacle;
- direct host-to-instrument communications;
- device identity access;
- capability discovery;
- status/diagnostic access;
- configuration access;
- normal firmware-update capability;
- a guaranteed physical firmware-recovery path;
- access to calibration information where appropriate.

If the project also claims **OI-02 — Open Instrument Protocol**, the direct host interface should expose the conceptual instrument model defined there.

## 3. Physical interface

The baseline connector is **USB-C** operating as a USB device/peripheral.

USB 2.0 Full Speed is sufficient for the baseline.

Higher USB speeds MAY be implemented where an instrument benefits from them, but a higher-speed USB requirement SHALL NOT be introduced merely for branding or convenience.

A typical USB 2.0 implementation includes:

- D+ / D−;
- CC1 / CC2 handling appropriate to the power/data role;
- VBUS sensing/power path;
- ESD/transient protection.

Exact connector mechanical placement is defined by OI-05 only for stack-compatible instruments.

## 4. Direct communications

A host SHALL be able to query and control the instrument directly through OI Standalone.

The transport framing is not yet frozen, but the semantics should map onto OI Protocol.

Illustrative operations:

```text
get_identity()
get_capabilities()
get_status()
get_measurements()
set_voltage(...)
set_mode(...)
arm(...)
```

A device MAY additionally expose human-friendly mechanisms such as USB CDC or SCPI-like text commands, provided those interfaces map onto the same underlying state and do not create conflicting control models.

## 5. Normal firmware update

A healthy instrument should provide a convenient software-initiated firmware-update path through OI Standalone.

Conceptually:

```text
application
    |
request update
    v
bootloader/update mode
    |
    v
new firmware
```

The image/container format, signing model, and update protocol are not frozen.

## 6. Required physical recovery for an OI Standalone claim

Every device claiming Draft 1 OI Standalone compatibility SHALL provide a dedicated physical **RECOVERY** control.

The canonical recovery procedure is:

```text
1. Power the instrument off and remove other instrument/service power.
2. Hold RECOVERY.
3. Connect OI Standalone USB-C to a powered host.
4. The device enters recovery/update mode directly.
5. Release RECOVERY.
6. Flash/reinstall firmware.
```

The intent is that this procedure remains recognizable across different reference and third-party instruments.

## 7. Recovery independence

Recovery SHALL NOT depend on the primary application firmware functioning correctly.

Acceptable implementation approaches include:

- an MCU ROM USB bootloader selected by hardware state;
- a protected first-stage bootloader that ordinary application update cannot erase.

The standard defines the required user-visible behavior, not a mandatory MCU or bootloader implementation.

## 8. Recovery power

USB VBUS SHALL power enough circuitry to perform recovery without any other power source.

The recovery path SHALL NOT require:

- OI Power;
- instrument-specific functional power;
- OI Link;
- OI Base;
- opening the enclosure;
- SWD/JTAG/debug probes;
- working application firmware.

The guaranteed recovery path should operate from ordinary USB-C 5 V VBUS and SHALL NOT depend on negotiating a higher USB-PD contract.

## 9. Recovery control requirements

The RECOVERY control SHOULD:

- be physically associated with OI Standalone;
- be clearly labelled;
- be protected against accidental activation;
- preferably be recessed or require deliberate operation;
- not be required for normal front-panel control.

Reference designs SHOULD NOT reuse RECOVERY as a normal user-interface button.

An additional software-triggered or powered long-hold recovery convenience MAY exist, but it does not replace the guaranteed power-off/hold/connect procedure.

## 10. USB-C as an optional instrument power source

OI Standalone may also provide operating power.

### 10.1 Low-power instruments

Low-power instruments SHOULD, where practical, support useful standalone operation from ordinary USB-C power.

### 10.2 USB Power Delivery

USB-PD support is optional.

An instrument MAY negotiate additional power over the same USB-C connector for partial or full functional operation.

Examples include:

- a larger display or compute load;
- a small programmable source;
- another instrument whose functional power envelope can adapt to the negotiated contract.

If functionality depends on the negotiated USB-PD contract, the currently available capability SHALL be reported through OI Protocol.

### 10.3 High-power instruments

A high-power instrument MAY use:

- USB only for service/control;
- USB-PD for some or all functional energy;
- a separate instrument-specific power input;
- OI Power for service electronics when stacked.

The instrument SHALL clearly report which functions are unavailable when the required energy source is missing.

## 11. Multiple power inputs

An OI Standalone implementation may coexist with OI Power and instrument-specific supplies.

The power architecture SHALL prevent prohibited backfeed between:

- USB VBUS;
- OI Power;
- instrument-specific power inputs.

Appearance/disappearance of one source SHOULD NOT unnecessarily reboot the service/control domain if another valid service source remains.

Detailed source-priority behavior is instrument-specific.

## 12. Grounding and isolation

Connecting OI Standalone SHALL NOT silently compromise the instrument's stated measurement/source isolation.

Every instrument claiming OI Standalone SHALL document the relationship between:

- USB ground;
- OI Power return where present;
- OI Link/control reference;
- chassis/shield;
- DUT/measurement ground;
- instrument-specific power return.

Where the instrument domain must remain isolated from the host, the implementation must provide appropriate isolation.

Connecting USB must not create an undocumented DUT-current path.

## 13. Host-facing USB profile

The exact canonical baseline USB class/framing is **TBD**.

Candidates include:

- USB CDC;
- vendor-specific bulk endpoints;
- HID-style control for very simple devices;
- another standard framing mapped onto OI Protocol.

The chosen baseline should favor:

- straightforward host support;
- cross-platform implementation;
- robust framing;
- easy discovery;
- low implementation burden;
- long-term maintainability.

## 14. Serviceability principle

An owner should always have a documented route from a non-booting instrument to known-good firmware using:

```text
USB-C cable
+ RECOVERY control
+ host computer
```

This is a serviceability/ownership guarantee, not merely a development feature.

## 15. Relationship to other standards

- **OI-01 — Open Instrument Link:** when also claimed, brings its required OI-02 Protocol semantics plus instrument-to-instrument communications and deterministic trigger.
- **OI-02 — Open Instrument Protocol:** when also claimed, provides the common conceptual instrument model exposed through OI Standalone.
- **OI-03 — Open Instrument Power:** when also claimed, provides optional/stack service power; it is never required for OI-04 recovery.
- **OI-05 — Open Instrument Stack:** when also claimed, fixes OI Standalone and RECOVERY locations on stack-compatible devices.

## 16. Draft 1 compliance intent

A Draft 1 OI Standalone implementation provides:

- USB-C;
- USB device communication;
- documented identity/capability access, using OI-02 semantics if OI-02 is also claimed;
- normal firmware update;
- dedicated physical RECOVERY control;
- application-independent recovery;
- USB-powered recovery from ordinary 5 V VBUS;
- documented grounding/isolation behavior.

USB-PD is an optional extension, not a baseline requirement.

## 17. Open decisions

Still open:

- canonical USB class/profile;
- host framing/serialization;
- firmware image/container format;
- update protocol;
- secure-boot/signing requirements;
- USB VID/PID strategy;
- standard host-side recovery tooling;
- whether optional PD capability needs a common descriptor beyond OI-02 power reporting.

## 18. Draft history

### Draft 1 — 2026-08-16

First consolidated OI-04 draft, including the guaranteed 5 V USB recovery rule and explicit optional USB-PD power role.
