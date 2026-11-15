# The emitter guide: drawing a model through Etassia

A model tool — a schema documentation generator, a UML repository browser, a
standards pipeline — draws through Etassia in three steps: it walks its own
object model, emits a DIM document, and hands the document to the engine.
The tool keeps its parser; the engine does the layout and the ink. This
guide walks the middle step, the DIM emission, with a small XML Schema as
the worked example.

## Step 1: walk the model, not the file

The emitter reads the model it already holds. For an XML Schema corpus that
is the Schema Object Model of lutaml-xsd; for a UML repository it is the
XMI model of ea; for EXPRESS it is the lutaml-express repository. The
emitter never reads the raw `.xsd`/`.xmi`/`.exp` file itself, because the
model has already resolved the semantics the file only implies — includes,
redefinitions, type unification.

## Step 2: decide the formation

A diagram is a decision about what deserves to be in the drawing. The
worked example below draws the type graph of one complex type: the type
itself as a node with its members as content blocks, the types it derives
from and references as further nodes, and the references as edges. The same
discipline applies to every family: the emitter answers which nodes, which
edges, and which layout direction tells the story — `DOWN` for type
hierarchies, `RIGHT` for message flows.

## Step 3: emit DIM

Each model element becomes a DIM node with `content` blocks; each
relationship becomes an edge with a semantic `style`; the document carries
the `notation` identifier of the family and the layout options. The engine
computes every position; the emitter never sets an `x` or a `y`.

### The worked example

The source schema:

```xml
<xs:complexType name="AddressType">
  <xs:sequence>
    <xs:element name="street" type="xs:string"/>
    <xs:element name="city" type="xs:string"/>
  </xs:sequence>
</xs:complexType>

<xs:complexType name="PersonType">
  <xs:sequence>
    <xs:element name="name" type="xs:string"/>
    <xs:element name="address" type="AddressType"/>
  </xs:sequence>
</xs:complexType>
```

The DIM document the emitter produces:

```json
{
  "dim": "0.1",
  "notation": "xsd",
  "graph": {
    "id": "root",
    "layoutOptions": { "elk.algorithm": "layered", "elk.direction": "DOWN" },
    "children": [
      {
        "id": "type-PersonType",
        "width": 200, "height": 110,
        "content": [
          { "kind": "label", "text": "PersonType", "role": "title" },
          { "kind": "label", "text": "name: xs:string", "role": "member" },
          { "kind": "label", "text": "address: AddressType", "role": "member" }
        ]
      },
      {
        "id": "type-AddressType",
        "width": 200, "height": 110,
        "content": [
          { "kind": "label", "text": "AddressType", "role": "title" },
          { "kind": "label", "text": "street: xs:string", "role": "member" },
          { "kind": "label", "text": "city: xs:string", "role": "member" }
        ]
      },
      {
        "id": "type-xs-string",
        "width": 160, "height": 50,
        "content": [
          { "kind": "label", "text": "xs:string", "role": "title" }
        ]
      }
    ],
    "edges": [
      {
        "id": "ref-person-address",
        "sources": ["type-PersonType"], "targets": ["type-AddressType"],
        "style": "association",
        "label": { "text": "address" }
      },
      {
        "id": "ref-address-street",
        "sources": ["type-AddressType"], "targets": ["type-xs-string"],
        "style": "reference"
      },
      {
        "id": "ref-address-city",
        "sources": ["type-AddressType"], "targets": ["type-xs-string"],
        "style": "reference"
      }
    ]
  }
}
```

## Step 4: hand it to the engine

The engine consumes the document over its C ABI as
`etassia_render_dim`, or through the `dim` notation of the registry. The
call returns the SVG, and the drawing is a function of the document and the
engine's version triple — the same document always draws the same bytes, so
the tool can pin, diff, and republish its diagrams like any other build
artifact.

## The styling boundary

The emitter carries no colors, fonts, or pixel choices. The semantic edge
styles (`association`, `generalization`, `composition`, `dependency`,
`reference`) and the content roles (`title`, `member`, `stereotype`,
`note`) are the entire styling vocabulary, and the renderer maps them to
its theme. One document therefore draws consistently in every theme, and a
theme change never touches an emitter.
