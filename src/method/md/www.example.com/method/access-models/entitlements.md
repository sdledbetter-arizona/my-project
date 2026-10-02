---
template:
  id: http://www.example.com/method/access-models/entitlements
  name: "Entitlements"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Entitlements



```table-editor
---
columns: { this: { label: "Entitlements" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix base: <http://www.example.com/method/base#> .
@prefix entitlement: <http://www.example.com/method/entitlement#> .

entitlement:EntitlementShape
    a sh:NodeShape ;
    sh:targetClass entitlement:Entitlement ;
    sh:property [
        sh:path base:itemName ;
        sh:name "Name" ;
        dash:editor dash:TextAreaEditor ;
        sh:maxCount 1 ;
    ] ;
    sh:property [
        sh:path base:itemDescription ;
        sh:name "Description" ;
        dash:editor dash:TextAreaEditor ;
        sh:maxCount 1 ;
    ] ;
    .
```

