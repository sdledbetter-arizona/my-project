# SailPoint Identity Governance Ontology

This repository contains an [OML](https://www.modelware.io/) ontology for a
SailPoint Identity Security Cloud identity governance and administration
system. The model covers identities, accounts, account correlation,
entitlements, roles, access profiles, and application sources relationships.

The ontology focuses on the conceptual elements managed by SailPoint ISC. HR
business processes, application-specific permission logic, organizational
politics, and user behavior are outside the ontology boundary.

## Repository layout

The repository uses separate OML vocabularies and descriptions:

```text
src/
	method/oml/www.example.com/method/vocabulary.oml
		Domain vocabulary: concepts, scalar properties, relations, and constraints.
	model/oml/www.example.com/project/description.oml
		Example description: named identities, accounts, entitlements, roles,
		application sources, and access profiles.
build/owl/
	Generated OWL/Turtle files produced by the export command.
```

The vocabulary defines the types and rules of the system. The description
provides concrete instances of those types. For example, an `Account` may be
linked to an `Identity`, hold entitlements, and belong to an
`ApplicationSource`; an `AccessProfile` belongs to an application source and
bundles entitlements from that source.

## Getting started

1. Open this folder in VS Code with the **OML Code** extension installed.
2. Use Git Bash in the repository root.
3. Run `oml lint` to check the OML files.
4. Run `oml export -o build/owl` to generate OWL/Turtle files.
5. Open the generated files under `build/owl/` in an OWL tool such as Protégé
	 when you need to inspect inferred classifications with a reasoner.

## Commands

Run these commands from Git Bash in the repository root:

```bash
oml start
oml lint
oml export -o build/owl
oml reason
```


- `oml start` starts the OML server.-
- `oml lint` checks the OML model for syntax and consistency errors.
- `oml export -o build/owl` exports the OML model as OWL/Turtle files.
- `oml reason` checks logical consistency and detects contradictions.
