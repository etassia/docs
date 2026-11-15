# etassia/docs

The specifications and architecture documents of the Etassia program.

Etassia is a diagram-rendering engine: it reads diagram sources — written in
a syntax family such as Mermaid or PlantUML, or derived from a model such as
XML Schema, JSON Schema, UML, or EXPRESS — and produces byte-identical SVG
from a deterministic pipeline, on every runtime it ships to. This repository
holds the documents that define that pipeline and its contracts.

## Contents

| Path | What it defines |
|---|---|
| [ARCHITECTURE.md](ARCHITECTURE.md) | The vision, the four layers, the pipeline, and the repository map |
| [dim/SPEC.md](dim/SPEC.md) | The Diagram Interchange Model 0.1: the boundary between model-holding tools and the engine |
| [dim/dim-0.1.schema.json](dim/dim-0.1.schema.json) | The DIM 0.1 JSON Schema (draft-07) |
| [dim/examples/](dim/examples/) | Worked DIM documents |
| [notations/](notations/) | The status page of each syntax family the engine carries |
| [emitter-guide.md](emitter-guide.md) | How a model tool emits DIM: a worked XML Schema example |

## Working copies

The engine's working copy of the DIM specification lives at
`docs/dim/` in `etassia/etassia-rs`, beside its implementation
(`crates/etassia-dim`). The two move together: a change to the
specification lands in this repository first, and the implementation
carries the same revision in its next release.
