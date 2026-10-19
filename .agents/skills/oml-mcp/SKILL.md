---
name: oml-mcp
description: "Use the OML MCP tools for this SailPoint Identity Governance ontology: identities, accounts, correlation, entitlements, roles, access profiles, application sources, SPARQL, updates, validation, and reasoning."
---

# OML MCP for this workspace

Use MCP as the primary interface for model lookup, querying, updates, validation, and reasoning. The repository models SailPoint Identity Security Cloud identity-governance concepts; do not substitute examples or assumptions from unrelated OML domains.

## Load tool schemas first

Before the first OML MCP call in a turn, use `tool_search_tool` to load the required OML tools. Batch the tools needed for the task in one search, for example:

`oml_workspace oml_search oml_about oml_members oml_paths oml_sparql oml_shapes oml_update oml_validate oml_reason oml_ontologies`

Use the appropriate returned `oml-*` tools. Do not call an OML tool whose schema is not loaded.

## Workspace vocabulary map

The `http://www.example.com/project/bundle` description bundle is the normal context for queries spanning project data. `oml_workspace` provides its current root IRI and the other roots.

| Prefix | IRI stem | Meaning |
|---|---|---|
| `base` | `http://www.example.com/method/base#` | `Identified`, `itemName`, `itemDescription`, `id`, and shared `Component` |
| `identity` | `http://www.example.com/method/identity#` | `Identity`, `identityType`, `HumanIdentity`, `MachineIdentity`, `AgentIdentity` |
| `account` | `http://www.example.com/method/account#` | `Account`, `nativeIdentity`, `isCorrelated`, `correlatesTo`, `holdsEntitlement`, `OrphanAccount` |
| `application_source` | `http://www.example.com/method/application-source#` | `ApplicationSource`, `hasAccount`, `hasEntitlement` |
| `entitlement` | `http://www.example.com/method/entitlement#` | `Entitlement` |
| `access_profile` | `http://www.example.com/method/access-profile#` | `AccessProfile`, `belongsToApplicationSource`, `bundles` |
| `role` | `http://www.example.com/method/role#` | `Role`, `containsEntitlement`, `containsAccessProfile` |
| `identities` | `http://www.example.com/project/identity-management/identities#` | Identity instances |
| `accounts` | `http://www.example.com/project/identity-management/accounts#` | Account instances |
| `application_sources` | `http://www.example.com/project/connections/application-sources#` | Application source instances |
| `entitlements` | `http://www.example.com/project/access-models/entitlements#` | Entitlement instances |
| `access_profiles` | `http://www.example.com/project/access-models/access-profiles#` | Access profile instances |
| `roles` | `http://www.example.com/project/access-models/roles#` | Role instances |

Use the exact predicate and class IRIs from the model/tools; this table is a navigation aid, not permission to invent axioms.

## Model-specific interpretation

- Human, Machine, and Agent are values of `identity:identityType`. Existing instances are declared as `identity:Identity` and carry values such as `"Human"`; do not assume the named subclasses are the asserted instance type.
- `base:itemName` is the normal display name. Accounts also have an application-specific `account:nativeIdentity`; do not confuse it with the display name or the correlated identity.
- Accounts belong to a source through `application_source:hasAccount` (inverse: `account`-side `isAccountOf`), correlate to identities through `account:correlatesTo`, and may hold entitlements through `account:holdsEntitlement`.
- An account is orphaned when its correlation state says it is uncorrelated (`account:isCorrelated false`, or the `account:OrphanAccount` class), not merely because a query fails to find a source link. The correlation flag and identity link must agree.
- Entitlement names and source names are `base:itemName` values. Entitlements are related to sources with `application_source:hasEntitlement`.
- An access profile belongs to one application source and may bundle zero or more entitlements. An empty profile has no `access_profile:bundles` assertions. Bundled entitlements must belong to that same source.
- Roles may contain entitlements and/or access profiles. The vocabulary does not define a role-to-identity assignment; entitlement overlap is not proof that an identity was assigned a role.
- The project does not consistently use `rdfs:label` for names. Query `base:itemName` unless the source specifically defines another naming property.

## Planning and retrieval

Determine whether the task is retrieval, update, or verification. Batch independent MCP calls where practical. Trust a tool result and do not repeat it unless a later update could have changed that result.

For retrieval, follow **search → about → (members or paths if needed) → SPARQL**:

1. Search once with a short domain term using `oml_search`. If no results, try one different short term; if both fail, ask what member or concept the user means. Search returns member types as well as matches.
2. Use `oml_about` on relevant IRIs in one call, with the project bundle as context for broad coverage. It often supplies enough asserted and inferred neighborhood information to answer without SPARQL.
3. If needed, use `oml_paths` to find relation paths between known members, or `oml_members` to enumerate a vocabulary's members. Use `oml_ontologies(includeImports=true)` only for questions about import structure.
4. Use `oml_sparql` for filtered, aggregated, or multi-hop data. Use a root from `oml_workspace` (normally the project description bundle) for full import closure coverage. Use predicates and types identified from the model; avoid guessing.

Query patterns for this model:

```sparql
# Include application sources that have no accounts.
PREFIX app: <http://www.example.com/method/application-source#>
PREFIX base: <http://www.example.com/method/base#>
SELECT ?source ?name (COUNT(DISTINCT ?account) AS ?accountCount)
WHERE {
  ?source a app:ApplicationSource .
  OPTIONAL { ?source base:itemName ?name }
  OPTIONAL { ?source app:hasAccount ?account }
}
GROUP BY ?source ?name
ORDER BY ?name
```

```sparql
# Find accounts explicitly marked uncorrelated.
PREFIX acct: <http://www.example.com/method/account#>
PREFIX base: <http://www.example.com/method/base#>
SELECT ?account ?name ?nativeIdentity
WHERE {
  ?account a acct:Account ; acct:isCorrelated false .
  OPTIONAL { ?account base:itemName ?name }
  OPTIONAL { ?account acct:nativeIdentity ?nativeIdentity }
}
ORDER BY ?name
```

To report who holds an entitlement, traverse `account:correlatesTo` from the account and `account:holdsEntitlement` to the entitlement; obtain both display names with `base:itemName`. Distinguish an account from its correlated identity in the answer.

Only if an expected SPARQL query returns empty, run one diagnostic probe against the exact predicate (`SELECT ?s ?p ?o WHERE { ?s <predicate-IRI> ?o } LIMIT 5`). An empty probe points to a predicate/coverage mismatch; a non-empty probe points to query logic. An empty result that itself answers the question needs no probe.

## Updating ontology data

Use `oml_update`, not direct edits to `.oml` files, for routine model changes.

1. Retrieve the relevant class, properties, current instances, and target description ontology. Target the existing project description that owns that kind of data rather than creating a new ontology.
2. Call `oml_shapes` on each class of an instance to create or modify. Treat `sh:targetClass`, every `sh:property/sh:path`, `sh:class`, and `sh:minCount` as the update contract. Use exact path IRIs for assertions. Check linked instances against the expected class. If no applicable shape/path is returned, inspect the vocabulary and ask rather than inventing a predicate.
3. Plan one atomic `oml_update` call for the complete change:
   - Create the instance, assert its RDF type, and add all required shape facts in the same call.
   - Add relationships only after confirming their exact paths and linked target types.
   - For a correlated `Account`, include `nativeIdentity`, `isCorrelated true`, and `correlatesTo`; also link it to exactly one application source with `hasAccount`. Do not add entitlements unless requested.
   - For an `AccessProfile`, assert its name and source; omit `bundles` when the user asks for an empty profile.
   - For an `Identity`, use the `Identity` class and set `identityType` to the requested controlled string value.
   - Routine single-instance additions can be applied directly. Use `preview: true` for removals, bulk changes, or unfamiliar update patterns when reviewing generated text first is valuable.
4. Avoid duplicate instance IDs and preserve existing user changes. Remove dependent links before deleting a member. Do not call `addImport` on a description ontology; imports belong to the appropriate bundle.
5. Address any lint problems reported by `oml_update`. Do not make speculative follow-up changes.
6. Once the update is lint-free, run `oml_validate` with `only: true` and `oml_reason` with `only: true, explanation: false` together. If reasoning reports inconsistency, request `explanation: true` before deciding on a fix.

## Verification and response

Report the concrete change or answer, the validation/reasoning outcome when relevant, and any limitation in the modeled facts. A role-to-identity assignment cannot be claimed because this vocabulary has no such relation. For source ownership or identity correlation, rely on the asserted link or its logically entailed inverse and distinguish that evidence from a name match.
