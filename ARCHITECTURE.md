# The Etassia architecture

## The vision

Etassia exists so that every formation a person can describe — as diagram
text or as a model — can be drawn as a diagram: laid out by deterministic
engines, rendered to the same bytes on every runtime, and reproducible for
as long as the source survives.

Mermaid and PlantUML happen to be the syntax families the engine carries
today. They are instances, not the definition. The definition is the layer
beneath them, and this document describes that layer.

## The pipeline

Every diagram, whatever its origin, travels the same road:

```
 source text                model (XSD, JSON Schema, UML, EXPRESS)
      |                            |
      v                            v
 notation registry           model tool (LutaML gem) emits DIM
 (claim + parse per family)        |
      |                            |
      +------------+---------------+
                   v
        Diagram Interchange Model (DIM)
                   |
                   v
        layout engine (elkr: ELK algorithms in Rust)
                   |
                   v
        renderer (per-kind ink: nodes, edges, markers, labels)
                   |
                   v
        SVG bytes, identical on every runtime
```

The stages are held together by one contract: **the same input, at the same
versions of every stage, always produces the same bytes.** The engine
reports the version triple that produced a drawing — (document, layout
engine, renderer) — and the byte-identity guarantee holds within one
triple. A change to any member of the triple is a rendering change and is
released as such.

## The four layers

### 1. The notation registry — syntax families are plug-ins

The registry is the engine's front door. A notation is any object that
claims a source, parses it, and names the diagram types it carries; the
Mermaid and PlantUML families are members, and a new family is a
registration rather than a change to the engine. Text families are parsed
by the engine's own grammar artifacts (compiled ahead of time and
checksummed, so parsing does not drift between deployments).

Model-derived diagrams do not enter as parsers. The authoritative parser
of a schema family stays where the model lives — lutaml-xsd for XML
Schema, lutaml-jsonschema for JSON Schema, ea for UML/XMI, lutaml-express
for EXPRESS — and the tool that holds the model emits a DIM document
(section 2). Etassia stays parser-thin by design: two parsers claiming the
same standard would be two opportunities to disagree.

### 2. The Diagram Interchange Model — the boundary

DIM (specified in [dim/SPEC.md](dim/SPEC.md)) is the engine's interchange
surface: a declarative graph — nodes with content, edges with semantic
intent — as an ELK JSON superset. It is the compiler-IR pattern applied to
diagrams: frontends (text notations, model-tool emitters) target one IR;
the backend (layout + render) evolves independently of all of them. The
emitter guide ([emitter-guide.md](emitter-guide.md)) walks through a
schema-to-DIM conversion step by step.

### 3. The layout engine — elkr

Layout is its own engine, not a private helper: [etassia/elkr](https://github.com/etassia/elkr)
is a Rust implementation of the Eclipse Layout Kernel algorithms, ported
from the pure-Ruby metanorma/elkrb. The Layered (Sugiyama) algorithm ships
first; force-directed placement and the remaining ELK algorithms follow the
roadmap in its README. Layout is deterministic by construction and
versioned with the engine, because the position of every node belongs to
the reproducibility contract as much as the stroke of every edge.

### 4. The delivery boundary — one ABI, every runtime

The engine exposes three functions — `etassia_render`, `etassia_detect`,
`etassia_free` — plus `etassia_render_dim` for DIM documents, over one C
ABI. Ruby reaches it through the compiled `etassia-native` gem, Python
through ctypes, JavaScript through WebAssembly, Rust through the crate.
Every runtime is first-class; no ecosystem is asked to keep a JavaScript
or Java sidecar alive to draw a diagram.

## The repository map

| Repository | Layer | What lives there |
|---|---|---|
| etassia/etassia | reference | The pure-Ruby reference implementation; every release is validated against it |
| etassia/etassia-grammar | notation | The grammar sources, compiled to the checksummed parse artifacts |
| etassia/etassia-rs | engine | The Rust runtime, the notation registry, `etassia-dim`, the C ABI (`etassia-ffi`), the wasm build, and the DIM specification's working copy |
| etassia/elkr | layout | The ELK layout algorithms in Rust |
| etassia/etassia-py | delivery | The Python package (ctypes over the ABI) |
| etassia/etassia-ts | delivery | The npm package (WebAssembly over the ABI) |
| etassia/etassia.github.io | site | www.etassia.org, with the playground running the wasm build |
| etassia/docs | documents | This repository: specifications and architecture |
| claricle/svgpng | adjacent | SVG-to-PNG rasterization with the same provenance discipline |

## What the vision deliberately excludes

Etassia is not a canvas editor: the input is text or a model, and the
output is ink. It does not chase novel layout for its own sake: layout
algorithms serve stability, because a layout that changes between versions
breaks the contract. And it does not silently reimplement every legacy
quirk of incumbent tools: parity is claimed only where a corpus proves it,
diagram type by diagram type.
