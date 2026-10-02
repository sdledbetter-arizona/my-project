---
template:
  id: http://www.example.com/method/identity-management/accounts
  name: "Accounts"
  rank: 0
  expose:
    - kind: compose
  params:
    - id: ontology
      type: iri
      defaultValue: ${context.ontology}
      required: true
---
# Accounts



```table-editor
---
columns: { this: { label: "Accounts" } }
---
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix dash: <http://datashapes.org/dash#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix base: <http://www.example.com/method/base#> .
@prefix account: <http://www.example.com/method/account#> .

account:AccountShape
    a sh:NodeShape ;
    sh:targetClass account:Account ;
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
        sh:path account:nativeIdentity ;
        sh:name "Native Identity" ;
        dash:editor dash:TextAreaEditor ;
    ] ;
    sh:property [
        sh:path account:isCorrelated ;
        sh:name "Is Correlated?" ;
        dash:editor dash:BooleanSelectEditor ;
        sh:datatype xsd:boolean ;
        sh:minCount 1 ;
        sh:maxCount 1 ;
    ] ;
    sh:property [
        sh:path account:correlatesTo ;
        sh:name "Identity" ;
    ] ;
    sh:property [
        sh:path account:holdsEntitlement ;
        sh:name "Entitlments" ;
    ] ;
    sh:sparql [
        a sh:SPARQLConstraint ;
        sh:message "The correlation flag must match whether this account links to an identity." ;
        sh:select """
            SELECT $this
            WHERE {
                {
                    $this <http://www.example.com/method/account#isCorrelated> true .
                    FILTER NOT EXISTS {
                        $this <http://www.example.com/method/account#correlatesTo> ?identity .
                    }
                }
                UNION
                {
                    $this <http://www.example.com/method/account#isCorrelated> false .
                    $this <http://www.example.com/method/account#correlatesTo> ?identity .
                }
            }
        """ ;
    ] ;
    .
```

## Business Rules
* The correlation flag must match the account's identity link: correlated accounts require exactly one identity, while uncorrelated accounts must not link to an identity.

