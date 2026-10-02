---
ontology: http://www.example.com/project/bundle
template:
  id: http://www.example.com/method/analysis/identity-governance-dashboard
  name: "Identity Governance Analysis Dashboard"
  rank: 0
  expose:
    - kind: compose
---
# Identity Governance Analysis Dashboard

## Application Access — Table

Shows accounts and their correlated identities and entitlements by application source. Uncorrelated accounts remain visible with no identity value.

```table
PREFIX base: <http://www.example.com/method/base#>
PREFIX account: <http://www.example.com/method/account#>
PREFIX application: <http://www.example.com/method/application-source#>

SELECT ?ApplicationName ?AccountName ?Correlated ?IdentityName ?EntitlementName WHERE {
    ?Application application:hasAccount ?Account ;
        base:itemName ?ApplicationName .
    ?Account base:itemName ?AccountName .
    OPTIONAL { ?Account account:isCorrelated ?Correlated . }
    OPTIONAL {
        ?Account account:correlatesTo ?Identity .
        ?Identity base:itemName ?IdentityName .
    }
    OPTIONAL {
        ?Account account:holdsEntitlement ?Entitlement .
        ?Entitlement base:itemName ?EntitlementName .
    }
}
ORDER BY ?ApplicationName ?AccountName ?EntitlementName
```

## Accounts by Application — Chart

Counts linked accounts for each source. Sources without linked accounts appear in the gap scan below.

```chart
---
type: bar
data:
  labels: ApplicationName
  datasets:
    - label: Accounts
      data: AccountCount
      backgroundColor: "#2878a5"
canvasHeight: "280px"
chartOptions:
  plugins:
    legend:
      display: false
  scales:
    y:
      beginAtZero: true
---
PREFIX base: <http://www.example.com/method/base#>
PREFIX application: <http://www.example.com/method/application-source#>

SELECT ?ApplicationName (COUNT(DISTINCT ?Account) AS ?AccountCount) WHERE {
    ?Application a application:ApplicationSource ;
        base:itemName ?ApplicationName ;
        application:hasAccount ?Account .
}
GROUP BY ?ApplicationName
ORDER BY ?ApplicationName
```

## Access Trace — Graph

Connects identities, accounts, held entitlements, and role content. A shared entitlement connects an account to a role's definition; it does not establish that the identity was assigned that role.

```graph
---
layout:
  mode: force
  fit: true
  padding: 32
group:
  byPredicate: true
---
PREFIX account: <http://www.example.com/method/account#>
PREFIX access: <http://www.example.com/method/access-profile#>
PREFIX role: <http://www.example.com/method/role#>

CONSTRUCT {
    ?Account account:correlatesTo ?Identity .
    ?Account account:holdsEntitlement ?Entitlement .
    ?Role role:containsEntitlement ?Entitlement .
    ?Role role:containsAccessProfile ?Profile .
    ?Profile access:bundles ?Entitlement .
}
WHERE {
    ?Account account:correlatesTo ?Identity ;
        account:holdsEntitlement ?Entitlement .
    OPTIONAL { ?Role role:containsEntitlement ?Entitlement . }
    OPTIONAL {
        ?Role role:containsAccessProfile ?Profile .
        ?Profile access:bundles ?Entitlement .
    }
}
```

## Computed Impact and Coverage

The following computation intersects each identity's held entitlements with each role's direct or access-profile entitlements, then calculates unique entitlement traceability coverage.

```python
role_entitlements = {
    "Systems Administrator": {"Entitlement1", "Entitlement5"},
    "HR Analyst": {"Entitlement4"},
}

identity_entitlements = {
    "Sheldon Ledbetter": {"Entitlement1", "Entitlement4", "Entitlement5", "Entitlement6", "Entitlement7"},
    "Bryan Swiger": {"Entitlement7"},
    "Dominic Beswick": {"Entitlement6"},
}

held_entitlements = set().union(*identity_entitlements.values())
role_trace = set().union(*role_entitlements.values())
traceable = held_entitlements & role_trace
untraced = held_entitlements - role_trace

print("Potential identity impact by current entitlement overlap:")
for role_name, grants in role_entitlements.items():
    affected = sorted(
        identity
        for identity, held in identity_entitlements.items()
        if held & grants
    )
    print(f"- {role_name}: {', '.join(affected) if affected else 'no current overlap'}")

coverage = 100 * len(traceable) / len(held_entitlements) if held_entitlements else 0
print(f"Role-traceable unique entitlements: {len(traceable)}/{len(held_entitlements)} ({coverage:.0f}%)")
print(f"Held entitlements without a role path: {', '.join(sorted(untraced)) or 'none'}")
```

## Gap Scan — Identities

Find identities missing the required identity type.

```table
PREFIX base: <http://www.example.com/method/base#>
PREFIX identity: <http://www.example.com/method/identity#>

SELECT ?Identity ?Name WHERE {
    ?Identity a identity:Identity ;
        base:itemName ?Name .
    FILTER NOT EXISTS { ?Identity identity:identityType ?type . }
}
```

## Gap Scan — Accounts



### Accounts that are orphans

```table
PREFIX base: <http://www.example.com/method/base#>
PREFIX account: <http://www.example.com/method/account#>

SELECT ?Account ?Name WHERE {
    ?Account a account:Account ;
        base:itemName ?Name ;
        account:isCorrelated false .
}
ORDER BY ?Name
```

### Accounts Without an Application Source

```table
PREFIX base: <http://www.example.com/method/base#>
PREFIX account: <http://www.example.com/method/account#>
PREFIX application: <http://www.example.com/method/application-source#>

SELECT ?Account ?Name WHERE {
    ?Account a account:Account ; base:itemName ?Name .
    FILTER NOT EXISTS { ?Source application:hasAccount ?Account . }
}
ORDER BY ?Name
```

### Accounts Without a Native Identity

```table
PREFIX base: <http://www.example.com/method/base#>
PREFIX account: <http://www.example.com/method/account#>

SELECT ?Account ?Name WHERE {
    ?Account a account:Account ; base:itemName ?Name .
    FILTER NOT EXISTS { ?Account account:nativeIdentity ?NativeIdentity . }
}
ORDER BY ?Name
```

### Accounts Without a Correlation Flag

```table
PREFIX base: <http://www.example.com/method/base#>
PREFIX account: <http://www.example.com/method/account#>

SELECT ?Account ?Name WHERE {
    ?Account a account:Account ; base:itemName ?Name .
    FILTER NOT EXISTS { ?Account account:isCorrelated ?Correlated . }
}
ORDER BY ?Name
```

### Correlated Accounts Without an Identity

```table
PREFIX base: <http://www.example.com/method/base#>
PREFIX account: <http://www.example.com/method/account#>

SELECT ?Account ?Name WHERE {
    ?Account a account:Account ;
        base:itemName ?Name ;
        account:isCorrelated true .
    FILTER NOT EXISTS { ?Account account:correlatesTo ?Identity . }
}
ORDER BY ?Name
```

## Gap Scan — Entitlements

Find entitlements with no source.

```table
PREFIX base: <http://www.example.com/method/base#>
PREFIX application: <http://www.example.com/method/application-source#>
PREFIX entitlement: <http://www.example.com/method/entitlement#>

SELECT DISTINCT ?Entitlement ?Name ?Gap WHERE {
    ?Entitlement a entitlement:Entitlement ; base:itemName ?Name .
    FILTER NOT EXISTS { ?Source application:hasEntitlement ?Entitlement . }
}
ORDER BY ?Name
```

### Entitlements Mapped to Multiple Sources


```table
PREFIX base: <http://www.example.com/method/base#>
PREFIX application: <http://www.example.com/method/application-source#>

SELECT DISTINCT ?Entitlement ?EntitlementName ?SourceName WHERE {
    ?Source1 application:hasEntitlement ?Entitlement ;
        base:itemName ?SourceName .
    ?Source2 application:hasEntitlement ?Entitlement .
    ?Entitlement base:itemName ?EntitlementName .
    FILTER (?Source1 != ?Source2)
}
ORDER BY ?EntitlementName ?SourceName
```

## Gap Scan — Access Profiles


### Access Profiles Without Assigned Application Source

```table
PREFIX base: <http://www.example.com/method/base#>
PREFIX access: <http://www.example.com/method/access-profile#>

SELECT ?Profile ?Name WHERE {
    ?Profile a access:AccessProfile ; base:itemName ?Name .
    FILTER NOT EXISTS { ?Profile access:belongsToApplicationSource ?Source . }
}
ORDER BY ?Name
```

### Access Profiles Without Bundled Entitlements

```table
PREFIX base: <http://www.example.com/method/base#>
PREFIX access: <http://www.example.com/method/access-profile#>

SELECT ?Profile ?Name WHERE {
    ?Profile a access:AccessProfile ; base:itemName ?Name .
    FILTER NOT EXISTS { ?Profile access:bundles ?Entitlement . }
}
ORDER BY ?Name
```

## Gap Scan — Roles

Find role definitions with neither direct entitlements nor access profiles.

```table
PREFIX base: <http://www.example.com/method/base#>
PREFIX role: <http://www.example.com/method/role#>

SELECT ?Role ?Name WHERE {
    ?Role a role:Role ;
        base:itemName ?Name .
    FILTER NOT EXISTS {
        { ?Role role:containsEntitlement ?Entitlement . }
        UNION
        { ?Role role:containsAccessProfile ?Profile . }
    }
}
ORDER BY ?Name
```

## Gap Scan — Application Sources


### Sources Without Accounts

```table
PREFIX base: <http://www.example.com/method/base#>
PREFIX application: <http://www.example.com/method/application-source#>

SELECT ?Source ?Name WHERE {
    ?Source a application:ApplicationSource ;
        base:itemName ?Name .
    FILTER NOT EXISTS { ?Source application:hasAccount ?Account . }
}
ORDER BY ?Name
```

### Sources Without Entitlements

```table
PREFIX base: <http://www.example.com/method/base#>
PREFIX application: <http://www.example.com/method/application-source#>

SELECT ?Source ?Name WHERE {
    ?Source a application:ApplicationSource ;
        base:itemName ?Name .
    FILTER NOT EXISTS { ?Source application:hasEntitlement ?Entitlement . }
}
ORDER BY ?Name
```

## Gap Scan — User Entitlement Traceability

Find correlated user entitlements with no route to either a direct role or a role's bundled access profile.

```table
PREFIX base: <http://www.example.com/method/base#>
PREFIX account: <http://www.example.com/method/account#>
PREFIX access: <http://www.example.com/method/access-profile#>
PREFIX role: <http://www.example.com/method/role#>

SELECT DISTINCT ?IdentityName ?AccountName ?EntitlementName WHERE {
    ?Account account:correlatesTo ?Identity ;
        account:holdsEntitlement ?Entitlement ;
        base:itemName ?AccountName .
    ?Identity base:itemName ?IdentityName .
    ?Entitlement base:itemName ?EntitlementName .
    FILTER NOT EXISTS {
        { ?Role role:containsEntitlement ?Entitlement . }
        UNION
        { ?Role role:containsAccessProfile ?Profile . ?Profile access:bundles ?Entitlement . }
    }
}
ORDER BY ?IdentityName ?EntitlementName
```