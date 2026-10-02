---
template:
  id: http://www.example.com/method/access-models/roles
  name: "Roles"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Roles



```table-editor
---
columns: { this: { label: "Roles" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix base: <http://www.example.com/method/base#> .
@prefix role: <http://www.example.com/method/role#> .

role:RoleShpe
    a sh:NodeShape ;
    sh:targetClass role:Role ;
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
        sh:path role:containsEntitlement ;
        sh:name "Entitlements" ;
    ] ;
    sh:property [
        sh:path role:containsAccessProfile ;
        sh:name "Access Profile" ;
    ] ;
    .
```

