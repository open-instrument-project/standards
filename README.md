# Open Instrument Standards

`SPDX-License-Identifier: Apache-2.0`

This repository contains the interoperability standards developed by the [Open Instrument Project](https://openinstrument.dev).

The standards are intentionally web-first and Markdown-native. They define opt-in interfaces for instruments that choose to communicate, coordinate, receive service power, provide direct host access, or share a common stackable mechanical form factor. Their small number of dependencies is explicit in the compatibility model below.

## Read, discuss, and contribute

The standards are public early so that interoperability decisions can be tested against real instruments and reviewed before they are frozen.

- Use [**Ideas & Proposals**](https://github.com/open-instrument-project/community/discussions/categories/ideas-proposals) for early technical ideas, cross-standard proposals, and changes that still need exploration.
- Use [**Project & Organisation**](https://github.com/open-instrument-project/community/discussions/categories/project-organisation) for standards process, roadmap, licensing, governance, and project-wide questions.
- Open an Issue in this repository when work is concrete and actionable.
- Open a pull request when there is a specific text, diagram, or asset change to review.

Small corrections and obvious fixes can go directly to a pull request. Please discuss changes that affect interoperability, public interfaces, electrical or mechanical compatibility, protocol semantics, licensing, or project-wide conventions before investing heavily in an implementation.

Before participating, read the project-wide [contribution policy](https://github.com/open-instrument-project/community/blob/main/CONTRIBUTING.md) and [Code of Conduct](https://github.com/open-instrument-project/community/blob/main/CODE_OF_CONDUCT.md) maintained in the `community` repository.

## Draft 1

All five standards in this repository are currently:

```text
Version: 0.1.0
Status: Draft 1
Date: 2026-08-16
```

They are design drafts, not yet stable interoperability guarantees. Sections explicitly marked **TBD**, **provisional**, **candidate**, or **not frozen** remain open.

| ID | Standard | Scope |
|---|---|---|
| [OI-01](OI-01-Link/index.md) | Open Instrument Link | CAN-FD communications using OI Protocol plus deterministic hardware trigger over commodity twisted-pair cabling |
| [OI-02](OI-02-Protocol/index.md) | Open Instrument Protocol | Transport-independent identity, capability, control, telemetry, fault, trigger, and composition model |
| [OI-03](OI-03-Power/index.md) | Open Instrument Power | 12 V common service/housekeeping power for compatible instruments and stacks |
| [OI-04](OI-04-Standalone/index.md) | Open Instrument Standalone | Direct USB-C host interface and guaranteed firmware recovery for instruments adopting this standard |
| [OI-05](OI-05-Stack/index.md) | Open Instrument Stack | Common mechanical format requiring OI Link, OI Protocol, and OI Power for chainable communication, triggering, semantics, and service power |

## Opt-in compatibility model

Open Instrument compatibility is opt-in:

- no OI standard is mandatory for every project;
- OI-02 Protocol, OI-03 Power, and OI-04 Standalone may be selected independently;
- OI-01 Link is a compound claim and requires OI-02 Protocol;
- OI-05 Stack is a compound claim and requires OI-01 Link, OI-02 Protocol, and OI-03 Power;
- normative requirements apply only when a project claims compatibility with that specific standard;
- where two standards describe how they interact, those rules apply only to projects claiming both;
- an unchecked standard means “not claimed”, not “failed”.

Projects report compatibility using an **OI Compatibility Badge** with a separate checkbox for each standard, for example:

```text
OI COMPATIBILITY
[x] OI-01 Link
[x] OI-02 Protocol
[ ] OI-03 Power
[x] OI-04 Standalone
[ ] OI-05 Stack
```

Project documentation should identify the revision of each standard it claims.

Selecting OI-01 on the badge therefore also requires the OI-02 checkbox to be selected. Selecting OI-05 requires the OI-01, OI-02, and OI-03 checkboxes to be selected.

## Core architecture

The standards deliberately separate several concerns:

```text
OI Standalone
    one instrument <-> one host

OI Link
    instrument <-> instruments
    OI Protocol over CAN-FD says what
    hardware trigger says when

OI Power
    service / housekeeping power
    not a universal DUT-energy bus

OI Stack
    mechanical packaging
    no proprietary electrical backplane

OI Protocol
    common instrument semantics across transports
```

The standards are modular. Implementations should claim only the standards they actually satisfy. A large rack instrument, for example, may implement OI Link and its required OI Protocol without adopting OI Power, OI Standalone, or OI Stack. A stack-compatible instrument necessarily implements OI Link, OI Protocol, and OI Power, while OI Standalone remains optional.

The reference hardware developed by the Open Instrument Project is expected to exercise these standards, but reference implementation choices such as MCU family, regulator IC, enclosure construction, or firmware architecture are not themselves requirements of the standards.

## Repository layout

```text
.
├── README.md
├── LICENSE
├── OI-01-Link/
│   ├── index.md
│   ├── assets/
│   └── published/
│       └── vX.Y.Z/
│           ├── index.md
│           └── assets/
├── OI-02-Protocol/
│   ├── index.md
│   └── assets/
├── OI-03-Power/
│   ├── index.md
│   └── assets/
├── OI-04-Standalone/
│   ├── index.md
│   └── assets/
└── OI-05-Stack/
    ├── index.md
    └── assets/
```

Markdown is the source of truth. Diagrams, mechanical drawings, reference pinout figures, and other standard-specific material belong in the corresponding `assets/` directory.

`index.md` and `assets/` are the current editable sources. A `published/` directory is added to a standard when its first revision is published; it contains immutable, self-contained snapshots rather than another working tree.

## Document identity, names, and versions

Each standard has a stable two-digit identifier:

```text
OI-01
OI-02
OI-03
OI-04
OI-05
```

The identifier does not change when the document is revised.

Standard directories use the identifier followed by the title-cased format name:

```text
OI-01-Link
OI-02-Protocol
OI-03-Power
OI-04-Standalone
OI-05-Stack
```

The identifier is authoritative. The format name is a readable path component and changes only if a standard is renamed deliberately. References SHALL use the exact path casing shown above so that links work consistently on case-sensitive hosts.

Technical document versions use semantic versioning independently for each standard:

- **PATCH** — clarification or erratum with no intended interoperability change;
- **MINOR** — backward-compatible addition or extension;
- **MAJOR** — incompatible normative change.

Before `1.0.0`, compatibility may still change as the standards mature.

Document maturity is separate from the semantic version. Intended maturity states are:

```text
Working Draft -> Draft -> Candidate -> Stable -> Deprecated
```

`Draft 1` in the current documents means the first consolidated Draft snapshot. The YAML front matter records `status: Draft` and `draft: 1` separately.

## Working copies and publication

`main` contains the latest editable working version of the suite.

Within each standard directory:

- `index.md` is the current working copy;
- `assets/` contains assets used by that working copy;
- `published/vX.Y.Z/` contains a frozen, self-contained published revision, including the assets needed by that revision.

Here, **published** means any deliberately released snapshot, including a Draft or Candidate revision; maturity remains explicit in the document metadata. A public working draft that has not been released as a named version remains only in `index.md`.

Publication happens independently for each standard. To publish a revision:

1. Set the version, status, and date in the working document.
2. Copy the document and all referenced assets to `published/vX.Y.Z/`.
3. Verify that the snapshot is self-contained and that its metadata matches the directory name.
4. Commit the immutable snapshot. Its directory path and document metadata identify the published revision.

A published snapshot is immutable. Corrections are published as a new PATCH revision; incompatible normative changes require a new MAJOR revision. A later change to one standard does not require an artificial release of the others.

The working `index.md` may move ahead immediately after publication. Before substantive work begins on the next revision, update it to the next intended version and set the appropriate working maturity. It must state its actual maturity and version clearly so readers do not mistake it for the latest published or Stable revision.

## Website publication

The [Open Instrument Project](https://openinstrument.dev) website may render these Markdown sources directly.

Preferred canonical routes:

```text
/standards/OI-01-Link/
/standards/OI-02-Protocol/
/standards/OI-03-Power/
/standards/OI-04-Standalone/
/standards/OI-05-Stack/
```

Published historical versions should remain available at immutable routes such as:

```text
/standards/OI-01-Link/v0.1.0/
/standards/OI-01-Link/v0.2.0/
/standards/OI-01-Link/v1.0.0/
```

While no Stable revision exists, the default page may show the current working/draft revision with an explicit status banner. Once a Stable revision exists, the default page should normally show the latest Stable revision and link prominently to any newer Working Draft.

## Normative language

In the standards:

- **SHALL / MUST** indicate an intended requirement;
- **SHALL NOT / MUST NOT** indicate an intended prohibition;
- **SHOULD** indicates a strong recommendation that may have justified exceptions;
- **MAY** indicates an optional capability.

Because these documents are Draft 1, requirements in sections marked as open or not frozen may still change before Candidate or Stable status.

## Licensing

Except where a file states otherwise, the standards, this README, and other documentation in this repository are licensed under the **Apache License 2.0** (`Apache-2.0`).

Each Markdown source includes an SPDX marker:

```text
SPDX-License-Identifier: Apache-2.0
```

The full licence text is provided in [`LICENSE`](LICENSE).

Contributions are made under the applicable project licence and require a [Developer Certificate of Origin 1.1](https://developercertificate.org/) sign-off. See the canonical [project-wide contribution policy](https://github.com/open-instrument-project/community/blob/main/CONTRIBUTING.md).

Participation is governed by the canonical project-wide [Code of Conduct](https://github.com/open-instrument-project/community/blob/main/CODE_OF_CONDUCT.md), based on Contributor Covenant 3.0.

The Open Instrument Project name, logos, compatibility marks, and other branding are not granted by the documentation licence.
