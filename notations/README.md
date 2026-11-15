# Syntax family status

The notation registry carries the syntax families below. A family enters
the registry as a grammar compiled to a checksummed parse artifact
([etassia/etassia-grammar](https://github.com/etassia/etassia-grammar)) plus
a hand-ported renderer per diagram type, validated against the reference
implementation's golden corpus.

| Family | Notation id | Grammar | Native diagram types | Status |
|---|---|---|---|---|
| Mermaid | `mermaid` | [etassia/etassia-grammar/mermaid](https://github.com/etassia/etassia-grammar) | 22 (flowchart, sequence, class, state, ER, gantt, gitgraph, pie, quadrant, xychart, radar, treemap, packet, requirement, C4, and the small boxes) | carried by the shipped runtimes; the golden corpus holds 1,206 cases at byte-identical parity |
| PlantUML | `plantuml` | carried in the reference implementation's notation registry | activity and class notations, with parity batches ongoing | carried by the Ruby reference implementation at HEAD; the Rust runtime and the browser build follow |

The detector recognizes 31 Mermaid diagram keywords in total; the detected
types without a native renderer (`sankey-beta`, `error`,
`flowchart-elk`) report the Unported condition, which the bindings
translate into a fallback call to the reference implementation.

Model-derived families do not appear in this table, because they do not
enter through text grammars: a schema or model tool emits a DIM document
([the emitter guide](../emitter-guide.md)), and the engine draws it through
the same pipeline. The first emitter has shipped — `Lutaml::Xsd::DimExporter`
in lutaml-xsd emits the XML Schema family — and the families with emitters
next are JSON Schema, UML (from Enterprise Architect XMI), and EXPRESS-G.
