# IITP Event Catalog

**Discussion draft · 28 June 2026 · environment-, application-, and workload-level events
for the Shared Signals Framework**

This catalog summarizes all candidate IITP event types so they can be reviewed together.
IITP events describe the security state of an **environment** (tenant, domain, cloud
account, cluster, resource, or workload) rather than an individual user, device, or session
— those remain the domain of CAEP and RISC.

Events are organized into five thematic groups. Each event reuses the shared IITP claims
(`severity`, `confidence`, `recommended_actions`) and the reverted / resolved / restored
follow-up pattern in addition to the key claims listed below.

All event-type URIs use the base `https://schemas.openid.net/secevent/iitp/event-type/`.

**Status legend** — **In spec**: already defined in the [IITP 1.0
draft](openid-iitp-1_0.md) · **Proposed**: candidate for discussion.

## 1. Cloud & Environment Posture

*Control-plane state and exposure.*

| # | Event type | Status | Subject | What it signals | Key claims | Typical receiver action |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | `configuration-drift` | **In spec** | Environment / resource | Security config drifted from known-good — e.g. privilege explosion on a role | `drift_type` `resource` `added_privileges` `affected_principals` `drift_status` | Freeze non-admin access; restrict holders of poisoned role until reverted |
| 2 | `security-control-degraded` | Proposed | Environment | A protective control is down/degraded (EDR offline, WAF monitor-only, logging dropped) | `control_type` `control_status` `coverage_gap` | Compensate with stricter access policy until coverage returns |
| 3 | `critical-vulnerability-exposed` | Proposed | Environment / workload | A critical, actively exploited vulnerability is present and reachable | `vulnerability_id` `exploit_status` `exposure` `affected_components` | Virtual-patch; restrict exposure; prioritize remediation |

## 2. Active Threat — App & Tenant

*An ongoing campaign in progress.*

| # | Event type | Status | Subject | What it signals | Key claims | Typical receiver action |
| --- | --- | --- | --- | --- | --- | --- |
| 4 | `asset-under-attack` | **In spec** | Tenant / domain / app | An app or tenant is under an active campaign (password spray, named adversary) | `attack_type` `attack_status` `scope` `threat_actor` `indicators` `recommended_posture` | Raise detection sensitivity; step-up auth; restrict cross-app access |
| 5 | `data-exfiltration` | Proposed | Tenant / environment | Bulk or anomalous data egress at the environment level | `data_classification` `volume` `destination` `exfil_status` | Throttle/block egress; snapshot for forensics; raise DLP sensitivity |
| 6 | `lateral-movement` | Proposed | Source + target env / workload | Adversary moving between workloads, accounts, or environments | `source` `target` `technique` `path` | Segment network; tighten east-west policy; isolate along path |

## 3. Workload & Supply Chain

*Trust in deployed code.*

| # | Event type | Status | Subject | What it signals | Key claims | Typical receiver action |
| --- | --- | --- | --- | --- | --- | --- |
| 7 | `workload-compromise` | **In spec** | Workload (complex subject) | A deployed workload is compromised / supply-chain poisoned | `compromise_type` `trust_status` `workload` `affected_component` `indicators` | Revoke workload identity; block service-account & agent traffic; quarantine |
| 8 | `anomalous-workload-behavior` | Proposed | Workload | Runtime anomaly short of confirmed compromise (low-confidence precursor to #7) | `behavior_type` `baseline_deviation` `workload` `confidence` | Monitor; quarantine pending review; raise detection |

## 4. Trust & Identity Infrastructure

*Issuers, keys, secrets, anchors.*

| # | Event type | Status | Subject | What it signals | Key claims | Typical receiver action |
| --- | --- | --- | --- | --- | --- | --- |
| 9 | `federation-trust-compromise` | Proposed | Issuer / federation / signing key | IdP, federation link, or token-signing key is compromised (Golden SAML, stolen key) | `trust_object` `affected_issuer` `compromise_vector` `valid_from` | Stop accepting assertions from issuer; force global reauth; rotate keys |
| 10 | `secret-exposure` | Proposed | Resource / workload | A non-human secret (service key, API token, cert) was exposed or leaked | `secret_type` `exposure_location` `affected_resource` `rotation_required` | Revoke tokens minted from it; force rotation; block the identity |
| 11 | `trust-anchor-change` | Proposed | Environment | A CA, trust-store entry, or pinned key was added / removed / changed | `anchor_type` `change_type` `fingerprint` `expected` | Distrust certs chaining to it; re-pin; alert |

## 5. Response Coordination & Lifecycle

*Hygiene, not threats.*

| # | Event type | Status | Subject | What it signals | Key claims | Typical receiver action |
| --- | --- | --- | --- | --- | --- | --- |
| 12 | `containment-status-change` | Proposed | Environment / workload | Something was isolated, quarantined, or released (the "action taken" signal) | `containment_action` `scope` `initiated_by` | Stay in sync; avoid double-acting or premature lift of restrictions |
| 13 | `environment-lifecycle-change` | Proposed | Environment | An environment was created, cloned, suspended, or decommissioned | `lifecycle_event` `expected` `parent` | Update inventory; flag unexpected creation |

## Discussion notes

Group 1 covers control-plane posture; Group 2 ongoing campaigns (with #6 lateral-movement
bridging to the workload group); Group 3 trust in deployed code (#8 anomalous behavior is a
deliberate low-confidence tier that escalates into #7); Group 4 fills the trust /
non-human-identity gap that CAEP and RISC do not cover; Group 5 is coordination and
inventory hygiene rather than threat detection.

Several of these candidates overlap with the [WISE profile][wise] — notably #3, #7, #8,
#10, and #11. See the [IITP / WISE comparison](openid-iitp-wise-comparison.html) for a
per-event breakdown of where the two profiles collide and where each is unique.

[wise]: https://identitymonk.github.io/openid-wise/
