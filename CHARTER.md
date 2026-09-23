# Internet Infrastructure Threat Profile (IITP) of the Shared Signals Framework

**Proposed charter / work-item addendum · Draft 0.1 · 28 June 2026**

> This document is a **proposed charter** for a new profile to be developed within the
> existing OpenID Shared Signals Working Group. It follows the structure of the Shared
> Signals Working Group Charter and other OpenID Foundation working-group charters. The
> Internet Infrastructure Threat Profile (IITP) would be a new deliverable added to the
> Working Group's scope, complementing the existing CAEP and RISC profiles. It is
> presented for discussion and is not an OpenID Foundation publication.

## 1) Work item name

Internet Infrastructure Threat Profile (IITP) — an environment- and application-level
event profile of the Shared Signals Framework [SSF]. The work would be conducted within
the OpenID Shared Signals Working Group as a peer profile to the Continuous Access
Evaluation Profile (CAEP) and the Risk Incident Sharing and Coordination (RISC) profile.

## 2) Purpose

The purpose of this work item is to define a profile of the Shared Signals Framework that
allows cooperating Transmitters and Receivers to coordinate their response to security
conditions that exist at the level of the **environment** — a cloud account, tenant,
domain, cluster, resource, or deployed workload — rather than at the level of an
individual human or robotic subject.

The existing Shared Signals profiles model the security state of an individual subject.
CAEP signals changes to a session, credential, assurance level, or device. RISC signals
account-lifecycle events such as a hijacked or disabled account. Both assume the relevant
unit of risk can be enumerated as a user, device, session, or account. A growing class of
real-world incidents does not fit that assumption: the problem is not (yet) a specific
compromised user, but a compromised or threatened *environment* — a cloud control plane
whose entitlements have drifted, an application tenant under an active campaign, or a
deployed workload whose software supply chain has been poisoned. In these situations the
appropriate response is taken at the level of a tenant, domain, role, or workload, and
frequently affects many subjects at once.

IITP will let a detection provider (for example a Cloud Infrastructure Entitlement
Management product, a SIEM, an application's own threat detection, or a workload
runtime-security product) relay an environment-level detection to Receivers — Identity
Providers, Policy Decision Points, integrated SaaS applications, and service meshes — that
are positioned to attenuate access, raise their detection posture, or withdraw trust, in
order to minimize the blast radius of the threat. IITP is complementary to, and not a
replacement for, CAEP and RISC; the profiles are intended to be used together, with IITP
describing the environment-level cause and CAEP/RISC carrying out per-subject enforcement.

## 3) Scope & objectives

The scope of this work item includes:

- Define an extensible set of Shared Signals Framework event types that describe
  environment-, application-, and workload-level security conditions and the actions a
  Receiver may take in response.
- Define a subject representation suitable for environment-level subjects (tenants,
  domains, cloud accounts, clusters, resources, and workloads), reusing existing SSF
  subject identifier formats where possible and defining an `environment` subject format
  where they are insufficient.
- Define a common set of advisory, non-binding "recommended action" semantics so that
  Transmitters can suggest responses (for example freezing non-admin access, restricting
  holders of a poisoned role, raising detection posture, or revoking a workload's identity)
  while Receivers retain full authority over enforcement.
- Define follow-up / clearing semantics (for example reverted, resolved, restored) so
  Receivers can lift measures applied in response to an event.
- Provide privacy and security considerations specific to environment-level signals,
  including the denial-of-service risks created by broad, automated responses.
- Provide non-normative use cases and examples to support interoperable implementation.

### Out of scope

- Defining the detection mechanisms by which a Transmitter determines that a condition
  exists (for example the analytics inside a CIEM, EDR, or SIEM). IITP standardizes the
  *signal*, not the detection.
- Defining the transport and stream-management mechanisms, which are inherited unchanged
  from the Shared Signals Framework.
- Per-subject (user, device, session, credential, account) events already covered by CAEP
  and RISC. Where an IITP event would duplicate an existing CAEP or RISC event, the
  existing event is preferred.
- Mandating specific enforcement actions, orchestration, or remediation workflows at the
  Receiver.
- Issuing statements regarding the reputation or quality of a user (consistent with the
  parent Working Group charter).

### Coordination

The work will be conducted within the Shared Signals Working Group and will coordinate
with adjacent OpenID working groups and external bodies, including the AB/Connect, IPSIE,
and AuthZEN working groups, and with the IETF (SET, Subject Identifiers for Security Event
Tokens). Where event semantics overlap with threat-sharing standards such as STIX,
alignment of vocabulary will be considered.

### Removing overlap with WISE

Early drafts of IITP defined event types that overlap with the [Workload Identity in
Secure Environments (WISE) profile][wise] — notably around federation trust, non-human
credentials, trust anchors, and runtime workload compromise. During a 2026-09-23 working
session between the IITP and WISE proposers, the two profiles agreed to adopt a clean
layering rather than continue to define overlapping events.

The layering is:

- **WISE — the confederacy of running workloads and the interplay between them.** Events
  about what is happening *inside* the boxes: federation, non-human credentials, trust
  anchors.
- **IITP — the environment those workloads run in.** Events about the things that operate
  the boxes: cloud posture, configuration drift, and supply-chain integrity.

Under that layering, the following changes are applied to the IITP event catalog:

| Item (earlier IITP draft) | Disposition | Rationale |
| --- | --- | --- |
| `secret-exposure` | Removed from IITP; covered by WISE `credential-compromise`. | Non-human credentials live in WISE. |
| `trust-anchor-change` | Removed from IITP; covered by WISE (or IPSIE where the anchor governs an OIDC federation). | Federation-layer concern, not environment posture. |
| `federation-trust-compromise` | Removed from IITP; covered by WISE (or IPSIE). | Federation-layer concern. |
| `workload-compromise` | Renamed to `workload-compromised` in IITP, sharing a name with the equivalent WISE event. | The two profiles carry the same event under the same name from different Transmitters — IITP as a detection at the environment level (typically supply-chain-driven), WISE as an authoritative statement by the trust domain issuer. |

Where an IITP receiver needs the corresponding per-workload signal, it is expected to
consume the WISE event. The two profiles are intended to be used together, with IITP
describing the environment-level cause and WISE / CAEP / RISC carrying out per-workload or
per-subject enforcement. Coordination between the IITP and WISE editors is expected to
continue as both profiles mature.

[wise]: https://identitymonk.github.io/openid-wise/

## 4) Proposed specification

The group proposes the following Specification deliverable:

### OpenID Internet Infrastructure Threat Profile 1.0

An extensible profile defining environment- and application-level event types conforming
to the Shared Signals Framework, the `environment` subject format, shared event claims
(`severity`, `confidence`, `recommended_actions`, and the reverted/resolved/restored
follow-up pattern), and the associated privacy and security considerations. The base URI
for the event types would be `https://schemas.openid.net/secevent/iitp/event-type/`.

The profile is organized into four thematic groups of event types. **The event types
listed below are candidate events only.** They are illustrative of the kinds of
environment- and application-level conditions the profile may cover, and are explicitly
offered as a starting point for discussion. The Working Group is expected to add, remove,
rename, merge, split, and otherwise modify these candidates — and the groupings themselves
— as the work matures. Nothing in this list should be read as a committed deliverable.

| Group | Event type | Signals |
| --- | --- | --- |
| Cloud & Environment Posture | `configuration-drift` | Security config drifted from known-good (e.g. privilege explosion on a role) |
| Cloud & Environment Posture | `security-control-degraded` | A protective control is down or degraded (EDR offline, WAF in monitor-only, logging dropped) |
| Cloud & Environment Posture | `critical-vulnerability-exposed` | A critical, actively exploited vulnerability is present and reachable |
| Active Threat — App & Tenant | `asset-under-attack` | An app or tenant is under an active campaign (password spray, named adversary) |
| Active Threat — App & Tenant | `data-exfiltration` | Bulk or anomalous data egress at the environment level |
| Active Threat — App & Tenant | `lateral-movement` | Adversary moving between workloads, accounts, or environments |
| Workload & Supply Chain | `workload-compromised` | A deployed workload is compromised / supply-chain poisoned (shares a name with the equivalent WISE event; distinguished by Transmitter, not by name) |
| Workload & Supply Chain | `anomalous-workload-behavior` | Runtime anomaly short of confirmed compromise (precursor) |
| Response Coordination & Lifecycle | `containment-status-change` | Something was isolated, quarantined, or released (the "action taken" signal) |
| Response Coordination & Lifecycle | `environment-lifecycle-change` | An environment was created, cloned, suspended, or decommissioned |

Federation-, credential-, and trust-anchor-related events that appeared in earlier IITP
drafts have been moved to the WISE profile (or IPSIE, where an OIDC federation is
governed). See §3 "Removing overlap with WISE" above.

The Working Group may also produce non-normative supporting material, including an
implementer's guide and interoperability test vectors, and may contribute
environment-level requirements back to the core Shared Signals Framework where
appropriate.

## 5) Anticipated audience or users

- Cloud Infrastructure Entitlement Management (CIEM) and Cloud Security Posture Management
  (CSPM) vendors.
- Security Information and Event Management (SIEM) and Extended Detection and Response
  (XDR) platforms.
- Identity Providers and Policy Decision Points that act on environment-level risk.
- Application and Software-as-a-Service (SaaS) vendors that participate in shared-signal
  exchanges.
- Workload-runtime and container-security vendors, and service-mesh / workload-identity
  platforms.
- Enterprises operating cloud and Kubernetes environments under a zero-trust security
  model.
- Endpoint, network, and supply-chain-security vendors.
- Implementers of CAEP and RISC seeking to coordinate environment-level and per-subject
  responses.

## 6) Language

Work will be conducted in English.

## 7) Method of work

E-mail discussions on the Shared Signals Working Group mailing list, working-group
conference calls, and face-to-face meetings from time to time, combined with collaborative
development via the Working Group's public repository.

## 8) Basis for determining when the work is completed

Rough consensus and running code. The work will be completed once an Implementer's Draft
and ultimately a Final Specification consistent with this purpose and scope have been
through the OpenID Foundation process, including review by the membership and demonstrated
by running code in one or more proof-of-concept, interoperability event, or commercial
projects.

## 9) Background information

The three use cases below motivated this work item and map directly to the core event
types.

### 9.1 Configuration drift / privilege explosion

A detection provider such as a CIEM product observes configuration drift in a cloud
environment — specifically a privilege explosion, where an IAM role (such as an S3 bucket
administrator) is modified to include additional, powerful privileges (such as EKS
privileges). The Identity Provider or other parties should be informed to act, for example
by blocking all access except for administrators, or by blocking anyone holding the newly
"poisoned" role until the environment is reverted to a known-good state.

### 9.2 Application or tenant under active attack

An application or environment is actively being attacked — for example a password-spray
campaign against a Salesforce tenant, or the detection of an adversary such as
ShinyHunters — a concern that does not fit neatly into enumerating every user account in
existing risk models. The instance under attack needs to communicate the threat at the
domain/tenant level to other integrated applications (such as ServiceNow) so they can raise
their overall detection level. The goal is to relay a detection to supporting components to
minimize the blast radius.

### 9.3 Supply-chain compromise in a homegrown application

In an enterprise deployment, a supply-chain attack is detected in a homegrown application
running in a Kubernetes cluster, resulting in a poisoned and actively compromised app.
Systems should know not to trust the compromised app, leading to protective measures such
as blocking all OAuth and service-to-service traffic between service accounts, or
preventing agents from communicating with the app.

### Related work and liaison relationships

- OpenID Continuous Access Evaluation Profile (CAEP) 1.0
- OpenID RISC Profile Specification 1.0
- OpenID Shared Signals Framework Specification 1.0
- RFC 8417 — Security Event Token (SET)
- Subject Identifiers for Security Event Tokens (IETF)
- OASIS STIX / TAXII — Structured Threat Information Expression
- MITRE ATT&CK — adversary tactics and techniques (for indicator/technique vocabulary)
- CISA Known Exploited Vulnerabilities (KEV) catalog (for vulnerability references)
- SPIFFE / SPIRE — workload identity

## 10) Proposers

- Mike Kiser, SailPoint
- Ian Glazer, CrowdStrike

*Additional proposers to be added.*

## 11) Anticipated contributions

- "OpenID Internet Infrastructure Threat Profile 1.0" — initial working draft (provided as
  input).
- IITP Event Catalog — a grouped summary of all proposed event types for prioritization
  (provided as input).

All to be made accessible through the [OpenID Foundation's IPR
Policy](https://openid.net/intellectual-property/).

---

*Drafted as input to the OpenID Shared Signals Working Group. Modeled on the Shared Signals
Working Group Charter and the IPSIE Working Group Charter. Not an OpenID Foundation
publication.*
