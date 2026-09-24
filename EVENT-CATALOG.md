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

For each event's triggers, claims, Receiver responses, and an example SET, see the
[IITP Event Reference](iitp-event-reference.html).

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
| 7 | `workload-compromised` | **In spec** | Workload (complex subject) | A deployed workload is compromised / supply-chain poisoned | `compromise_type` `trust_status` `workload` `affected_component` `indicators` | Revoke workload identity; block service-account & agent traffic; quarantine |
| 8 | `anomalous-workload-behavior` | Proposed | Workload | Runtime anomaly short of confirmed compromise (low-confidence precursor to #7) | `behavior_type` `baseline_deviation` `workload` `confidence` | Monitor; quarantine pending review; raise detection |

> **Naming note:** IITP's `workload-compromised` deliberately shares a name with WISE's `workload-compromised`. Under the 2026-09-23 layering the two Transmitters describe distinct facts (IITP: an environment-level detection, typically supply-chain-driven; WISE: an authoritative statement by the trust domain issuer) but a receiver treats them as the same event.

## 4. Response Coordination & Lifecycle

*Hygiene, not threats.*

| # | Event type | Status | Subject | What it signals | Key claims | Typical receiver action |
| --- | --- | --- | --- | --- | --- | --- |
| 9 | `containment-status-change` | Proposed | Environment / workload | Something was isolated, quarantined, or released (the "action taken" signal) | `containment_action` `scope` `initiated_by` | Stay in sync; avoid double-acting or premature lift of restrictions |
| 10 | `environment-lifecycle-change` | Proposed | Environment | An environment was created, cloned, suspended, or decommissioned | `lifecycle_event` `expected` `parent` | Update inventory; flag unexpected creation |

## Discussion notes

Group 1 covers control-plane posture; Group 2 ongoing campaigns (with #6 lateral-movement
bridging to the workload group); Group 3 trust in deployed code (#8 anomalous behavior is
a deliberate low-confidence tier that escalates into #7); Group 4 is coordination and
inventory hygiene rather than threat detection.

### Relationship to WISE

IITP and the [WISE profile][wise] were previously overlapping on federation trust,
non-human credentials, trust anchors, and workload compromise. Following the 2026-09-23
working session between the two proposers, the profiles adopt a clean layering:

- **WISE = the confederacy of running workloads and the interplay between them** — events
  about what is happening *inside* the boxes. Federation, non-human credentials, and
  trust anchors live here.
- **IITP = the environment those workloads run in** — the things that operate the boxes.
  Cloud posture, configuration drift, and supply-chain integrity live here.

Under that layering, the following IITP candidates from earlier drafts have been removed
in favour of the equivalent WISE events: `federation-trust-compromise`, `secret-exposure`
(→ WISE `credential-compromise`), and `trust-anchor-change`. IITP's `workload-compromise`
event has been renamed `workload-compromised` (#7) to align tense with the WISE event of
the same name; the two profiles carry the same event under the same name from two
different Transmitters — IITP as a detection at the environment level (typically
supply-chain-driven), WISE as an authoritative statement by the trust domain issuer. See
the [IITP / WISE comparison](openid-iitp-wise-comparison.html) for the up-to-date
per-event breakdown.

[wise]: https://identitymonk.github.io/openid-wise/
