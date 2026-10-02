---
template:
  id: http://www.example.com/method/access-models/access-profiles
  name: "Access Profiles"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Access Profiles



```table-editor
---
columns: { this: { label: "Access Profiles" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix base: <http://www.example.com/method/base#> .
@prefix access-profile: <http://www.example.com/method/access-profile#> .
@prefix application-source: <http://www.example.com/method/application-source#> .

access-profile:AccessProfileShape
    a sh:NodeShape ;
    sh:targetClass access-profile:AccessProfile ;
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
        sh:path access-profile:belongsToApplicationSource ;
        sh:name "Source" ;
    ] ;
    sh:property [
        sh:path access-profile:bundles ;
        sh:name "Bundled Entitlements" ;
    ] ;
    sh:sparql [
      a sh:SPARQLConstraint ;
      sh:message "Every bundled entitlement must belong to the access profile's application source." ;
      sh:select """
        SELECT $this ?source ?entitlement
        WHERE {
          $this <http://www.example.com/method/access-profile#belongsToApplicationSource> ?source ;
            <http://www.example.com/method/access-profile#bundles> ?entitlement .
          FILTER NOT EXISTS {
            ?source <http://www.example.com/method/application-source#hasEntitlement> ?entitlement .
          }
        }
      """ ;
    ] ;
    .
```

  ## Business Rule
  Every entitlement bundled by an access profile must be provided by the same application source assigned to that profile.

