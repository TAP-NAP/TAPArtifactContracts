# TAP Artifact Contracts

[English](README.md) | [简体中文](README.zh-CN.md)

## Purpose

The shared specification for TAP capture applications, browser verifiers and
backend services. It defines product behavior, media encoding/decoding, manifest
fields, signed-byte coverage and HTTP interfaces. Implementations follow this
repository; their source and documentation do not supply missing contract rules.

## Usage

Start with the document for the question you need to answer:

| Document | Read it for |
| --- | --- |
| [Contract index](CONTRACTS.md) | Manifest fields, shared encoder/decoder layouts, hashes and signatures |
| [Product contract](ProductContract.md) | App behavior, lifecycle and claim boundaries |
| [Backend contract](BackendContract.md) | App Attest registration, HTTP, trust and replay policy |
| [Planes design](PlanesTechnicalDesign.md) | Photo geometry and its limits |
| [Acceptance](Acceptance.md) | Validation procedures and the evidence each result requires |
| [Examples and vectors](examples/README.md) | Synthetic fields and exact byte/transform cases |
| [Versioning](VERSIONING.md) | Independent format families and reviewed revisions |

There is no application to install or build. To edit the specification, check
local Markdown links and anchors and parse the JSON examples. With Python 3:

```sh
python3 - <<'PY'
import json
from pathlib import Path

for path in Path("examples").rglob("*.json"):
    json.loads(path.read_text(encoding="utf-8"))
print("JSON examples parsed")
PY
```

Parsing JSON checks syntax, not conformance. Preserve exact vectors; do not
regenerate expected bytes to match an implementation. Follow
[VERSIONING.md](VERSIONING.md) when changing behavior or bytes.

## How it works

Encoding and decoding describe how media blobs are written and read for playback
or analysis. The manifest describes the captured artifact. Binding and proof
rules define which stored bytes enter each hash and how the signed message is
reconstructed. Decoding success, depth quality and sample timing do not determine
whether the stored bytes match their hashes and signature.

Product and backend contracts define the surrounding behavior and service
interface. Acceptance procedures describe how to validate those requirements;
they are not claims that an implementation has passed.

## Directory structure

```text
.
├── CONTRACTS.md          # Entry point for artifact format and verification rules
├── ProductContract.md    # Product behavior and visible interaction requirements
├── BackendContract.md    # Shared HTTP, credentials and trust requirements
├── PlanesTechnicalDesign.md # Geometry algorithm and limitations
├── Acceptance.md         # Capability-specific validation procedures
├── VERSIONING.md         # Format versions and reviewed contract revisions
├── manifests/           # Still Photo, Live Photo and TAP Video metadata
├── containers/          # Photo/video byte layouts, depth and telemetry codecs
├── bindings/            # Hash inputs, signing messages, proof and verification
├── transport/           # Unsigned .tapnap package routing
└── examples/            # Synthetic documents and exact test vectors
    ├── manifests/       # Artifact metadata examples
    ├── transport/       # Package routing examples
    └── vectors/         # Fixed bytes and expected codec/transform results
```

## Repository dependencies

The documentation dependency is one-way:

```text
Capture applications ─┐
Browser verifiers ────┼──> TAPArtifactContracts
Backend services ────┘
```

This repository has no code or documentation dependency on those consumers.
Each consumer records its reviewed contract commit and owns its build, runtime
dependencies and implementation evidence. A reviewed Git revision is separate
from wire-format versions; editorial changes do not require a new consumer pin.
Generic libraries retain their own API documentation and need not depend on TAP
contracts unless they implement TAP-specific rules.
