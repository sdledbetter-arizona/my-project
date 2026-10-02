---
template:
  id: http://www.example.com/method/connections/application-sources
  name: "Application Sources"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Application Source



```table-editor
---
columns: { this: { label: "Application Source" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix base: <http://www.example.com/method/base#> .
@prefix application-source: <http://www.example.com/method/application-source#> .

application-source:ApplicationSourceShape
    a sh:NodeShape ;
    sh:targetClass application-source:ApplicationSource ;
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
    sh:property [
        sh:path application-source:hasAccount ;
        sh:name "Accounts" ;
    ] ;
    sh:property [
        sh:path application-source:hasEntitlement ;
        sh:name "Entitlements" ;
    ] ;
    .
```

