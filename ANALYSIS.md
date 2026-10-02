# Analysis Findings

This report maps the four methodology questions to evidence in the [Identity Governance Analysis Dashboard](src/model/md/SailPoint/Identity%20Governance/Analysis%20Dashboard.md). Its query blocks run against the project bundle; the script computation is based on the current example-instance snapshot.

## 1. Impact of Role or Entitlement Changes

**Question:** Which identities may be affected when a role definition changes or an entitlement is removed?

**Evidence:** The dashboard's **Access Trace — Graph** and **Computed Impact and Coverage** sections compare account-held entitlements with direct role entitlements and access-profile entitlements included by roles. The live overlap query currently returns three role-entitlement-identity rows, all for Sheldon Ledbetter: Domain Admin and Team_Admin under Systems Administrator, and Member_Only under HR Analyst.

**Finding:** Sheldon is a potential impact candidate for changes to those role grants because his accounts hold the matching entitlements. This does not prove that Sheldon was assigned either role. The model has no role-to-identity assignment relation and no change history or before/after role versions.

**Gap source:** Vocabulary - Add role assignment and effective-dated change evidence if confirmed role membership and historical impact are required.

## 2. Users with Access to Each Application

**Question:** What users have access to each application?

**Evidence:** The dashboard's **Application Access — Table** joins application sources to their accounts, correlated identities, and held entitlements. The chart counts linked accounts by source.

**Finding:** The current model returns 7 accounts across Active Directory (2), Dropbox (2), and DocuSign (3). Six accounts correlate to identities; Account2 (`bs0012`) is uncorrelated, so its owner cannot be identified from this model. LogicGate and MD-Staff currently have no linked accounts.

**Gap source:** Process - populate correlation information before attributing that access to a user.

## 3. Orphan Accounts

**Question:** Which accounts are orphans?

**Evidence:** The dashboard's **Gap Scan — Accounts** query selects accounts with `isCorrelated false` and also checks missing account-source, native-identity, correlation-flag, and identity links.

**Finding:** Account2 (`bs0012`) is the current orphan account. The other account relationship checks return no gaps in the current examples.

**Gap source:** None.

## 4. Entitlement Traceability

**Question:** Can every entitlement a user has be traced to a role, policy, or approval?

**Evidence:** **Gap Scan — User Entitlement Traceability** reports correlated account entitlements with no direct role or role-to-access-profile path. The computed analysis finds 3 of 5 unique entitlements held by correlated accounts have a role path (60%). The gap query returns four account-level rows for DocuSign Sender and DS Sender/Template Share.

**Finding:** No. DocuSign Sender and DS Sender/Template Share currently have no role path. The vocabulary has no Policy or Approval concepts or links, so those two trace routes cannot be tested at all. The entitlement source scan also finds Team_Admin mapped to both Dropbox and DocuSign despite the single-source modeling rule.

**Gap source:** Vocabulary and model data. Add policy/approval concepts and trace links if those are in scope; correct Team_Admin's source mapping.

## Scope and Method

The analysis follows asserted relationships in the current project bundle. The role-impact computation identifies entitlement overlap only; it is not a role assignment or historical change-impact calculation. Script inputs mirror current example data and should be updated when those descriptions change.