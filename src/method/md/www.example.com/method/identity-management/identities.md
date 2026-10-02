---
template:
  id: http://www.example.com/method/identity-management/identities
  name: "Identities"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Identities



```table-editor
---
columns: { this: { label: "Identities" } }
stylesheet:
  - selector: cell[col === "Type"  && value]
    target: value
    style:
      padding: 4px 12px
      border-radius: 999px
      font-size: 12px
      font-weight: 600
      color: "#7a6cff"
      background-color: "#ede9ff"
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix base: <http://www.example.com/method/base#> .
@prefix identity: <http://www.example.com/method/identity#> .

identity:IdentityShape
    a sh:NodeShape ;
    sh:targetClass identity:Identity ;
    sh:property [
        sh:path base:itemName ;
        sh:name "Name" ;
        dash:editor dash:TextAreaEditor ;
        sh:maxCount 1 ;
    ] ;
    sh:property [
        sh:path identity:identityType;
        sh:name "Type";
        sh:maxCount 1 ;
        sh:minCount 1 ;
    ];
    .
```

