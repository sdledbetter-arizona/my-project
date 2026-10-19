---
name: oml-text
description: "Use when inspecting or authoring this project's OML source text: SailPoint identity governance vocabularies and descriptions, OML syntax, aliases, imports, file layout, or source-level documentation."
---

# OML Text for this workspace

Use this skill when the task depends on OML source text, syntax, declarations, aliases, annotations, or file layout. For model navigation, live data queries, ordinary instance updates, SHACL validation, or reasoning, use the `oml-mcp` skill when its MCP tools are available. Do not edit generated files under `build/`.

## Project scope and file layout

This repository models SailPoint Identity Security Cloud identity governance: identities, accounts, account correlation, entitlements, roles, access profiles, and application sources.

```text
src/method/oml/www.example.com/method/
  base.oml
  identity.oml
  account.oml
  application-source.oml
  entitlement.oml
  access-profile.oml
  role.oml
  bundle.oml
src/model/oml/www.example.com/project/
  bundle.oml
  identity-management/identities.oml
  identity-management/accounts.oml
  connections/application-sources.oml
  access-models/entitlements.oml
  access-models/access-profiles.oml
  access-models/roles.oml
```

Vocabulary files define domain types and constraints; project description files define named example instances. The project description bundle at `http://www.example.com/project/bundle#` includes the project descriptions and is normally the root for model-wide MCP queries. The method bundle at `http://www.example.com/method/bundle#` groups the method vocabularies.

## Vocabulary and aliases

| Alias in descriptions | Vocabulary | Main members |
|---|---|---|
| `base` | `http://www.example.com/method/base#` | `Identified`, `itemName`, `itemDescription`, `id`, `Component` |
| `identity` | `http://www.example.com/method/identity#` | `Identity`, `identityType`, `HumanIdentity`, `MachineIdentity`, `AgentIdentity` |
| `account` | `http://www.example.com/method/account#` | `Account`, `nativeIdentity`, `isCorrelated`, `correlatesTo`, `holdsEntitlement`, `OrphanAccount` |
| `application` or `application-source` | `http://www.example.com/method/application-source#` | `ApplicationSource`, `hasAccount`, `hasEntitlement` |
| `entitlement` | `http://www.example.com/method/entitlement#` | `Entitlement` |
| `access-profile` | `http://www.example.com/method/access-profile#` | `AccessProfile`, `belongsToApplicationSource`, `bundles` |
| `role` | `http://www.example.com/method/role#` | `Role`, `containsEntitlement`, `containsAccessProfile` |

Aliases are local to each file. Follow the `uses`/`extends` declarations in that file instead of assuming an alias spelling.

## Domain patterns to preserve

- Every identity is an `identity:Identity` instance with exactly one `identity:identityType` value from `"Human"`, `"Machine"`, or `"Agent"`. Human/Machine/Agent identity concepts are defined by equivalence restrictions on that value; existing descriptions assert `Identity` and the scalar type value.
- `base:itemName` is the usual display label. `account:nativeIdentity` is the identifier used by the connected application and is not a replacement for the account's item name or correlated person's identity.
- `account:Account` requires one `nativeIdentity`, one `isCorrelated` boolean, and one application source (`application-source:isAccountOf`). The correlation flag must agree with the presence of `account:correlatesTo`: true means linked to an identity; false means no identity link. `account:OrphanAccount` is defined by `isCorrelated false`.
- Application-source descriptions own their account links through `application:hasAccount`; they also map available entitlements through `application:hasEntitlement`. An account's `holdsEntitlement` lists account grants. Keep these distinct.
- An `entitlement:Entitlement` belongs to exactly one application source. Access profiles use `access-profile:belongsToApplicationSource` and may list same-source entitlements with `access-profile:bundles`; no bundle assertions means the profile is empty.
- Roles contain entitlements directly or contain access profiles through `role:containsEntitlement` and `role:containsAccessProfile`. The vocabulary does not define assignment of roles to identities, so do not author or infer a role assignment.
- The vocabulary uses `base:itemName`, not a project-wide `rdfs:label` convention.

## Source authoring workflow

1. Identify whether the change belongs to a vocabulary under `src/method/oml/` or to example instance data under `src/model/oml/`.
2. Read the target file and its directly relevant imports before editing. Follow existing IRI, alias, namespace, indentation, and description conventions. Keep the model facts in the owning description file.
3. For an instance-data update, prefer `oml-mcp` and `oml_update`; do not directly edit data files when MCP is available. Direct source editing is appropriate when the task specifically concerns text, vocabulary definitions, or MCP is unavailable.
4. For an account/source/identity change, inspect the corresponding identities, accounts, and application-source descriptions together. Preserve both account correlation and source ownership semantics.
5. Keep updates minimal. Do not add a new vocabulary, predicate, role-assignment relation, or annotation convention unless the requirement calls for it and its implications are clear.
6. Validate direct edits with the repository's OML tooling (for example `oml lint` from the repository root); use MCP validation and reasoning for model-level verification when available.

## Source examples from this repository's syntax

Identity descriptions use `identity:Identity` and controlled text for `identityType`:

```oml
instance ExampleHuman : identity:Identity [
    base:itemName "Example Human"
    identity:identityType "Human"
]
```

Account descriptions separate native ID, correlation status, identity link, and entitlement grants:

```oml
instance ExampleAccount : account:Account [
    base:itemName "Example Human - Workday HR"
    account:nativeIdentity "example-human"
    account:isCorrelated true
    account:correlatesTo identities:ExampleHuman
]
```

The application-source description links the account to its source. Omitting `hasEntitlement` is valid when the source has none:

```oml
instance ExampleSource : application:ApplicationSource [
    base:itemName "Workday HR"
    application:hasAccount accounts:ExampleAccount
]
```

An empty access profile has a source link and no `bundles` assertion:

```oml
instance ExampleProfile : access-profile:AccessProfile [
    base:itemName "Example Workday Profile"
    access-profile:belongsToApplicationSource applications:ExampleSource
]
```

These are syntax patterns, not requests to add these particular example members. Check the actual imports and shapes before applying a new instance.

## Response expectations

When explaining source-level findings, distinguish what is explicitly written in source from what is inferred by the OML model. State cross-file assumptions when combining vocabulary constraints with description assertions.
