---
stand_alone: true
ipr: none
cat: std # Check
submissiontype: IETF
wg: OpenID Shared Signals

docname: openid-iitp-1_0

title: "OpenID Internet Infrastructure Threat Profile 1.0 - draft 01"
abbrev: IITP-Spec
lang: en
kw:
 - security events
 - shared signals
 - infrastructure
 - threat
 - SET

author:
- ins: M. Kiser
  name: Mike Kiser
  org: SailPoint
- ins: I. Glazer
  name: Ian Glazer
  org: CrowdStrike

normative:
  RFC2119:
  RFC5646:
  RFC8174:
  RFC8417:
  SSF:
    title: "OpenID Shared Signals Framework Specification 1.0"
    target: https://openid.net/specs/openid-sharedsignals-framework-1_0.html
    author:
      - ins: A. Tulshibagwale
        name: Atul Tulshibagwale
      - ins: T. Cappalli
        name: Tim Cappalli
      - ins: M. Scurtescu
        name: Marius Scurtescu
      - ins: A. Backman
        name: Annabelle Backman
      - ins: J. Bradley
        name: John Bradley
      - ins: S. Miel
        name: Shayne Miel
    date: 2025
  CAEP:
    title: "OpenID Continuous Access Evaluation Profile 1.0"
    target: https://openid.net/specs/openid-caep-1_0.html
    author:
      - ins: T. Cappalli
        name: Tim Cappalli
      - ins: A. Tulshibagwale
        name: Atul Tulshibagwale
    date: 2025
  RISC:
    title: "OpenID RISC Profile Specification 1.0"
    target: https://openid.net/specs/openid-risc-1_0.html
    author:
      - ins: M. Scurtescu
        name: Marius Scurtescu
      - ins: A. Backman
        name: Annabelle Backman
      - ins: P. Hunt
        name: Phil Hunt
      - ins: J. Bradley
        name: John Bradley
      - ins: S. Bounev
        name: Stefan Bounev
      - ins: A. Tulshibagwale
        name: Atul Tulshibagwale
    date: 2025

informative:
  IITP-EVENTS:
    title: "IITP Event Reference"
    target: https://derrumbe.github.io/openid-iitp/iitp-event-reference.html
    date: 2026
  SPIFFE:
    title: "Secure Production Identity Framework for Everyone (SPIFFE)"
    target: https://spiffe.io/

--- abstract

This document defines the Internet Infrastructure Threat Profile (IITP) of the Shared Signals Framework {{SSF}}. It specifies a set of event types conforming to the Shared Signals Framework. Where the Continuous Access Evaluation Profile (CAEP) {{CAEP}} and the RISC Profile {{RISC}} are primarily concerned with the state of individual human or robotic subjects -- users, devices, sessions, and credentials -- IITP is concerned with the state of the environment in which those subjects operate: cloud control planes, application tenants, domains, and deployed workloads.

These event types are intended to be used between cooperating Transmitters and Receivers so that a detection at the infrastructure or application level (for example a privilege explosion in a cloud role, an application tenant under active attack, or a supply-chain compromise of a deployed workload) can be relayed to other parties that are able to attenuate access, raise their detection posture, or withdraw trust in order to minimize the blast radius of the threat.

--- middle

# Introduction

IITP is the application of the Shared Signals Framework {{SSF}} to environment-level and application-level security coordination in a network of cooperating providers. IITP specifies a set of event types that conform to the SSF. This document specifies the event types required to achieve this goal.

Existing Shared Signals profiles model the security state of an individual subject. CAEP signals changes to a session, a credential, an assurance level, or a device's compliance. RISC signals account-level events such as a hijacked or disabled account. Both assume the interesting unit of risk can be enumerated as a user, device, session, or account. A growing class of real-world incidents does not fit that assumption cleanly: the problem is not (yet) a specific compromised user, but a compromised environment -- a cloud control plane whose entitlements have drifted, an application tenant under an active campaign, or a deployed workload whose software supply chain has been poisoned. In these situations the appropriate response is taken at the level of a tenant, a domain, a role, or a workload, and frequently affects many subjects at once.

IITP defines event types that let a detection provider (for example a Cloud Infrastructure Entitlement Management product, a Security Information and Event Management platform, an application's own threat detection, or a workload runtime security product) relay an environment-level detection to Receivers -- Identity Providers, Policy Decision Points, integrated SaaS applications, and service meshes -- that are positioned to act on it.

## Relationship to CAEP and RISC

IITP is complementary to, and not a replacement for, CAEP {{CAEP}} and RISC {{RISC}}. The profiles are intended to be used together. A typical coordinated response combines them: an IITP `configuration-drift` event may describe the environment-level cause, while the resulting CAEP `session-revoked` or `token-claims-change` events carry out the per-subject enforcement. Where an IITP event and a CAEP or RISC event would describe the same fact, Transmitters SHOULD prefer the existing CAEP or RISC event for the per-subject fact and use IITP only for the environment-level fact that the existing profiles cannot express.

## Scope and Subjects

The subject of an IITP event identifies the affected element of the environment rather than an individual human or robotic principal. Depending on the event type, the subject is typically a tenant, an organization unit, a domain, a cloud resource (such as an IAM role), or a deployed workload. Because the existing SSF subject formats are oriented toward users, devices, and sessions, this profile defines an additional optional subject format, `environment` ({{environment-subject}}), and also permits the `complex` subject format defined in {{SSF}} to combine an environment identifier with a more specific resource identifier.

## Notational Conventions

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

# Common Event Claims {#common-event-claims}

The following claims MAY appear in any IITP event unless an event definition specifies otherwise. They are defined once here and referenced from each event type. The semantics of `event_timestamp`, `initiating_entity`, `reason_admin`, and `reason_user` are aligned with the corresponding claims in CAEP {{CAEP}} so that an implementation already supporting CAEP can reuse its handling.

Attributes:

- **event_timestamp** - OPTIONAL, JSON number: the time at which the condition described by this SET was detected by the Transmitter. Its value is the number of seconds from 1970-01-01T0:0:0Z as measured in UTC. Because IITP events describe detections rather than authoritative state changes, this is the time of detection, which MAY be later than the time the underlying condition first arose.
- **initiating_entity** - OPTIONAL, JSON string: describes what produced the detection. This MUST be one of:
    - `detector` - an automated detection system (e.g., a CIEM, EDR, or SIEM) produced the event.
    - `policy` - a policy evaluation produced the event.
    - `admin` - an administrative action or manual investigation produced the event.
    - `system` - a platform or service assertion produced the event.
- **reason_admin** - OPTIONAL, JSON object: a localizable administrative message intended for logging and auditing. The object MUST contain one or more key/value pairs, with a BCP 47 {{RFC5646}} language tag as the key and the locale-specific message as the value.
- **reason_user** - OPTIONAL, JSON object: a localizable end-user-friendly message, structured identically to `reason_admin`. Because IITP events are environment-level, this claim is frequently omitted.
- **severity** - RECOMMENDED, JSON string: the Transmitter's assessment of the severity of the condition. This MUST be one of `low`, `medium`, `high`, or `critical`.
- **confidence** - OPTIONAL, JSON string: the Transmitter's confidence that the reported condition is real (as opposed to a false positive). This MUST be one of `low`, `medium`, or `high`. Receivers SHOULD weight automated, high-impact responses by this value.
- **recommended_actions** - OPTIONAL, JSON array of strings: an advisory, non-binding list of actions the Transmitter suggests the Receiver consider. Values are drawn from the registry in {{receiver-processing}}, or any other value mutually understood by the Transmitter and Receiver. A Receiver MUST NOT treat these as commands; the decision to act, and the action taken, remain entirely the Receiver's responsibility ({{receiver-processing}}).

The following example is non-normative.

~~~ json
{
  "reason_admin": {
    "en": "Privilege explosion on role arn:aws:iam::123456789012:role/s3-bucket-admin",
    "ja": "ロール s3-bucket-admin で権限が急増しました"
  }
}
~~~

# Event Types

The base URI for IITP event types is:

`https://schemas.openid.net/secevent/iitp/event-type/`

This document specifies three event types. Further candidate event types are under discussion and are not specified here. The IITP Event Reference {{IITP-EVENTS}} summarizes both the specified and the candidate event types, with their triggers, claims, Receiver responses, and example SETs.

## configuration-drift

Event Type URI: `https://schemas.openid.net/secevent/iitp/event-type/configuration-drift`

The `configuration-drift` event signals that the security-relevant configuration of an environment or of a resource within it has changed away from a known-good or expected state in a way that materially changes the environment's risk. The canonical case is a "privilege explosion": an entitlement (for example an IAM role such as an S3 bucket administrator) is modified to include additional, powerful privileges (for example cluster-administration privileges on a Kubernetes service), so that any principal holding that role now wields far broader access than intended.

The subject of this event identifies the affected resource and, where applicable, the environment that contains it ({{environment-subject}}). The Transmitter is typically a detection provider such as a Cloud Infrastructure Entitlement Management product and is not necessarily authoritative over the resource; Receivers SHOULD treat the event as a detection to be acted upon according to local policy.

Attributes:

- **drift_type** - REQUIRED, JSON string: the category of drift detected. This MUST be one of the following, or any other value mutually understood by the Transmitter and Receiver:
    - `privilege-escalation` - an identity, role, or policy gained additional privileges.
    - `policy-change` - a security policy (e.g., a guardrail or SCP) was weakened or removed.
    - `resource-exposure` - a resource was made more broadly accessible (e.g., a bucket made public).
    - `network-exposure` - a network control was weakened (e.g., a security group opened).
    - `encryption-change` - encryption was disabled or downgraded.
    - `logging-disabled` - audit logging or monitoring was disabled.
- **resource** - REQUIRED, JSON object: identifies the configuration item that drifted. The object MUST contain a `type` (JSON string, e.g. `iam-role`, `iam-policy`, `storage-bucket`, `security-group`) and an `id` (JSON string, e.g. a cloud resource name or ARN). It MAY contain `provider` (e.g. `aws`, `gcp`, `azure`), `region`, and `account` to disambiguate the resource.
- **added_privileges** - OPTIONAL, JSON array of strings: privileges, permissions, or scopes that were added by the drift. For a privilege explosion this is the set of newly granted, "poisoning" privileges.
- **removed_privileges** - OPTIONAL, JSON array of strings: privileges or controls that were removed by the drift.
- **affected_principals** - OPTIONAL, JSON object: describes the set of principals that hold or are affected by the drifted resource, so that Receivers can scope enforcement. It MAY contain `count` (JSON number) and `selector` (JSON string), where the selector is an expression mutually understood by the parties (for example the role name whose holders should be restricted).
- **baseline_ref** - OPTIONAL, JSON string: an opaque reference to the known-good baseline the resource has drifted from, enabling correlation with a later remediation or a "drift resolved" follow-up event.
- **drift_status** - OPTIONAL, JSON string: one of `detected`, `persisting`, or `reverted`. If omitted, `detected` is assumed. A Transmitter SHOULD send a follow-up event with `drift_status` set to `reverted` when the environment is restored to a known-good state, so that Receivers can lift any restrictions they applied.

When `event_timestamp` is included, its value MUST represent the time at which the drift was detected.

The following example is non-normative. It shows a privilege explosion on an IAM role, using an environment subject with optional claims.

~~~ json
{
  "iss": "https://ciem.example.com/",
  "jti": "f3a1c0de9b7e4a2f8c6d1e0b5a4f7c21",
  "iat": 1782691200,
  "aud": "https://idp.example.com/ssf",
  "txn": "drift-2026-0001",
  "sub_id": {
    "format": "environment",
    "environment_type": "cloud-account",
    "provider": "aws",
    "id": "123456789012"
  },
  "events": {
    "https://schemas.openid.net/secevent/iitp/event-type/configuration-drift": {
      "drift_type": "privilege-escalation",
      "resource": {
        "type": "iam-role",
        "provider": "aws",
        "account": "123456789012",
        "region": "us-east-1",
        "id": "arn:aws:iam::123456789012:role/s3-bucket-admin"
      },
      "added_privileges": [
        "eks:*",
        "eks:AccessKubernetesApi"
      ],
      "affected_principals": {
        "selector": "role:s3-bucket-admin",
        "count": 14
      },
      "baseline_ref": "baseline-2026-06-01T00:00:00Z",
      "drift_status": "detected",
      "severity": "high",
      "confidence": "high",
      "initiating_entity": "detector",
      "recommended_actions": [
        "restrict-role-holders",
        "freeze-non-admin-access"
      ],
      "reason_admin": {
        "en": "Role s3-bucket-admin modified to include EKS cluster-admin privileges; treat role as poisoned until reverted."
      },
      "event_timestamp": 1782691180
    }
  }
}
~~~

The following example is non-normative. It shows the follow-up event signalling that the drift has been reverted.

~~~ json
{
  "iss": "https://ciem.example.com/",
  "jti": "a91b2c3d4e5f60718293a4b5c6d7e8f9",
  "iat": 1782698400,
  "aud": "https://idp.example.com/ssf",
  "txn": "drift-2026-0001",
  "sub_id": {
    "format": "environment",
    "environment_type": "cloud-account",
    "provider": "aws",
    "id": "123456789012"
  },
  "events": {
    "https://schemas.openid.net/secevent/iitp/event-type/configuration-drift": {
      "drift_type": "privilege-escalation",
      "resource": {
        "type": "iam-role",
        "provider": "aws",
        "account": "123456789012",
        "id": "arn:aws:iam::123456789012:role/s3-bucket-admin"
      },
      "drift_status": "reverted",
      "baseline_ref": "baseline-2026-06-01T00:00:00Z",
      "severity": "low",
      "initiating_entity": "detector",
      "reason_admin": {
        "en": "Role reverted to known-good baseline; restrictions may be lifted."
      },
      "event_timestamp": 1782698380
    }
  }
}
~~~

## asset-under-attack

Event Type URI: `https://schemas.openid.net/secevent/iitp/event-type/asset-under-attack`

The `asset-under-attack` event signals that an application instance, tenant, domain, or other environment identified by the subject is currently the target of an active attack campaign. Unlike a per-user risk signal, this event describes a condition at the level of the asset as a whole -- for example a password-spraying campaign against a SaaS tenant, or the detection of a known adversary operating within a tenant. Its purpose is to let the asset under attack warn other integrated components (for example other SaaS applications connected to the same organization) so that they can raise their own detection posture and reduce the overall blast radius, even before any individual account at those components is known to be compromised.

Attributes:

- **attack_type** - REQUIRED, JSON string: the category of attack observed. This MUST be one of the following, or any other value mutually understood by the Transmitter and Receiver:
    - `password-spray`
    - `credential-stuffing`
    - `brute-force`
    - `account-takeover`
    - `mfa-fatigue`
    - `data-exfiltration`
    - `api-abuse`
    - `reconnaissance`
    - `adversary-presence` - a named or profiled adversary has been detected operating in the asset.
- **attack_status** - REQUIRED, JSON string: the lifecycle state of the attack as assessed by the Transmitter. This MUST be one of `suspected`, `active`, `contained`, or `resolved`. A Transmitter SHOULD send a follow-up event transitioning the status to `contained` or `resolved` so Receivers can relax any elevated posture.
- **scope** - RECOMMENDED, JSON string: the granularity at which the attack is being reported, so Receivers understand how broadly to apply any response. This MUST be one of `tenant`, `domain`, `application`, or `organization`.
- **threat_actor** - OPTIONAL, JSON object: when the Transmitter has attribution, describes the adversary. It MAY contain `name` (JSON string, e.g. a tracked actor name), `attribution_confidence` (one of `low`, `medium`, `high`), and `reference` (a URI to threat intelligence describing the actor).
- **indicators** - OPTIONAL, JSON array of objects: indicators of compromise associated with the campaign that a Receiver may use to tune its own detection. Each object SHOULD contain a `type` (e.g. `ip`, `asn`, `user-agent`, `domain`) and a `value`.
- **recommended_posture** - OPTIONAL, JSON string: a coarse advisory describing how the Transmitter suggests Receivers adjust their detection sensitivity, distinct from the concrete `recommended_actions` of {{common-event-claims}}. This MUST be one of `normal`, `heightened`, or `maximum`.

When `event_timestamp` is included, its value MUST represent the time at which the attack was detected or last observed.

The following example is non-normative. It shows a password-spray campaign against a SaaS tenant, warning an integrated application.

~~~ json
{
  "iss": "https://salesforce.example.com/",
  "jti": "7c2e9a1b6f3d4e5a8b0c1d2e3f405162",
  "iat": 1782700000,
  "aud": "https://servicenow.example.com/ssf",
  "txn": "attack-tenant-77",
  "sub_id": {
    "format": "environment",
    "environment_type": "tenant",
    "domain": "example.com",
    "id": "00D5e000000ABCdEAU"
  },
  "events": {
    "https://schemas.openid.net/secevent/iitp/event-type/asset-under-attack": {
      "attack_type": "password-spray",
      "attack_status": "active",
      "scope": "tenant",
      "severity": "high",
      "confidence": "high",
      "recommended_posture": "heightened",
      "indicators": [
        { "type": "asn", "value": "AS14061" },
        { "type": "user-agent", "value": "python-requests/2.31" }
      ],
      "recommended_actions": [
        "elevate-detection",
        "require-step-up-auth"
      ],
      "initiating_entity": "detector",
      "reason_admin": {
        "en": "High-volume password spray observed against tenant; integrated apps should raise detection sensitivity."
      },
      "event_timestamp": 1782699940
    }
  }
}
~~~

The following example is non-normative. It shows the detection of a profiled adversary within a tenant.

~~~ json
{
  "iss": "https://salesforce.example.com/",
  "jti": "1f0e2d3c4b5a69788796a5b4c3d2e1f0",
  "iat": 1782703600,
  "aud": "https://servicenow.example.com/ssf",
  "txn": "attack-tenant-78",
  "sub_id": {
    "format": "environment",
    "environment_type": "tenant",
    "domain": "example.com",
    "id": "00D5e000000ABCdEAU"
  },
  "events": {
    "https://schemas.openid.net/secevent/iitp/event-type/asset-under-attack": {
      "attack_type": "adversary-presence",
      "attack_status": "active",
      "scope": "tenant",
      "severity": "critical",
      "confidence": "medium",
      "recommended_posture": "maximum",
      "threat_actor": {
        "name": "ShinyHunters",
        "attribution_confidence": "medium",
        "reference": "https://threatintel.example.com/actors/shinyhunters"
      },
      "recommended_actions": [
        "elevate-detection",
        "restrict-cross-app-access"
      ],
      "initiating_entity": "detector",
      "reason_admin": {
        "en": "Activity consistent with ShinyHunters observed in tenant; minimize blast radius across connected apps."
      },
      "event_timestamp": 1782703540
    }
  }
}
~~~

## workload-compromise

Event Type URI: `https://schemas.openid.net/secevent/iitp/event-type/workload-compromise`

The `workload-compromise` event signals that a deployed workload -- an application, service, container, or function identified by the subject -- is believed to be compromised and should no longer be trusted. The motivating case is a supply-chain attack against a homegrown application deployed in a Kubernetes cluster: a poisoned dependency or build artifact results in an actively compromised running app. On receiving this event, relying systems can withdraw trust from the workload -- for example by blocking OAuth or service-to-service traffic to and from its service account, revoking its workload identity, or preventing agents from communicating with it -- in order to contain the compromise.

Attributes:

- **compromise_type** - REQUIRED, JSON string: the nature of the compromise. This MUST be one of the following, or any other value mutually understood by the Transmitter and Receiver:
    - `supply-chain` - a compromised dependency or third-party component.
    - `build-pipeline` - a compromised build or CI/CD process.
    - `malicious-artifact` - a known-bad image, package, or artifact was deployed.
    - `runtime-compromise` - the running workload exhibits compromise at runtime.
    - `secret-exposure` - the workload's secrets or identity material were exposed.
- **trust_status** - REQUIRED, JSON string: the trust the Transmitter asserts should now be placed in the workload. This MUST be one of `untrusted`, `quarantined`, or `restored`. A Transmitter SHOULD send a follow-up event with `trust_status` set to `restored` once the workload has been remediated and redeployed from a clean state.
- **workload** - REQUIRED, JSON object: identifies the affected workload. The object SHOULD identify the workload as precisely as the deployment allows and MAY contain: `name`, `namespace`, `cluster`, `image`, `image_digest` (e.g. a `sha256:` digest), `service_account`, and `spiffe_id` (a SPIFFE {{SPIFFE}} workload identifier, where present).
- **affected_component** - OPTIONAL, JSON object: identifies the specific component believed responsible, for example the poisoned dependency. It MAY contain `type` (e.g. `package`, `base-image`), `name`, and `version`.
- **indicators** - OPTIONAL, JSON array of objects: indicators associated with the compromise, structured as in the `asset-under-attack` event.

When `event_timestamp` is included, its value MUST represent the time at which the compromise was detected.

The following example is non-normative. It shows a supply-chain compromise of a Kubernetes workload, using a complex subject with optional claims.

~~~ json
{
  "iss": "https://runtime-sec.example.com/",
  "jti": "9b8a7c6d5e4f30211203a4b5c6d7e8f9",
  "iat": 1782710000,
  "aud": "https://mesh.example.com/ssf",
  "txn": "wl-compromise-42",
  "sub_id": {
    "format": "complex",
    "environment": {
      "format": "environment",
      "environment_type": "k8s-cluster",
      "id": "prod-cluster-eu-1"
    },
    "workload": {
      "format": "opaque",
      "id": "payments-api@prod"
    }
  },
  "events": {
    "https://schemas.openid.net/secevent/iitp/event-type/workload-compromise": {
      "compromise_type": "supply-chain",
      "trust_status": "untrusted",
      "workload": {
        "name": "payments-api",
        "namespace": "prod",
        "cluster": "prod-cluster-eu-1",
        "image": "registry.example.com/payments-api:1.8.3",
        "image_digest": "sha256:5e0c...a91f",
        "service_account": "payments-api-sa",
        "spiffe_id": "spiffe://example.com/ns/prod/sa/payments-api-sa"
      },
      "affected_component": {
        "type": "package",
        "name": "left-pad-utils",
        "version": "4.2.0"
      },
      "severity": "critical",
      "confidence": "high",
      "initiating_entity": "detector",
      "recommended_actions": [
        "revoke-workload-identity",
        "block-service-account-traffic",
        "block-agent-communication"
      ],
      "reason_admin": {
        "en": "Supply-chain compromise via poisoned dependency left-pad-utils@4.2.0; do not trust payments-api service account."
      },
      "event_timestamp": 1782709950
    }
  }
}
~~~

# Subjects and the Environment Subject Format {#environment-subject}

IITP events MAY use any subject format defined in {{SSF}}. Because IITP subjects are frequently environments or resources rather than users, this profile defines an additional optional subject format, `environment`, and recommends the `complex` format {{SSF}} when an event must identify both an environment and a more specific resource or workload within it.

The `environment` subject format has the following members:

| Member | Requirement | Description |
| --- | --- | --- |
| `format` | REQUIRED | The literal string `environment`. |
| `environment_type` | REQUIRED | JSON string indicating the kind of environment, e.g. `cloud-account`, `tenant`, `domain`, `k8s-cluster`, `org-unit`. |
| `id` | REQUIRED | JSON string uniquely identifying the environment within the scope of `environment_type` and any qualifying members. |
| `provider` | OPTIONAL | JSON string identifying the platform or vendor (e.g. `aws`, `gcp`, `azure`, a SaaS vendor). |
| `domain` | OPTIONAL | JSON string giving the DNS domain associated with the environment, useful for correlating a tenant across applications. |

A Receiver that does not understand the `environment` subject format MUST treat the event as it would any other event with an unrecognized subject and SHOULD NOT act on it. Transmitters SHOULD only send IITP events to Receivers that have indicated support for this profile through the Shared Signals Framework stream configuration {{SSF}}.

# Receiver Processing and Recommended Actions {#receiver-processing}

IITP events are detections, not commands. A Transmitter conveys what it observed and MAY suggest a response through the `recommended_actions` claim, but the Receiver retains full authority over whether and how to respond. This separation is deliberate: the Receiver is best positioned to understand the operational impact of, for example, freezing non-administrative access to an environment, and is responsible for avoiding self-inflicted denial of service.

The following values are defined for the `recommended_actions` claim. Receivers MAY support any subset, and Transmitters and Receivers MAY agree on additional values.

| Value | Meaning |
| --- | --- |
| `freeze-non-admin-access` | Block all access to the affected environment except for administrators. |
| `restrict-role-holders` | Restrict access for principals holding a specific (drifted/"poisoned") role until reverted. |
| `elevate-detection` | Raise detection sensitivity / lower alerting thresholds for the affected scope. |
| `require-step-up-auth` | Require additional authentication for access to the affected scope. |
| `restrict-cross-app-access` | Tighten or suspend cross-application access associated with the affected tenant or domain. |
| `revoke-workload-identity` | Revoke or suspend the credentials / identity of the affected workload. |
| `block-service-account-traffic` | Block OAuth or service-to-service traffic to and from the affected workload's service account. |
| `block-agent-communication` | Prevent agents or other services from communicating with the affected workload. |
| `quarantine` | Isolate the affected resource or workload pending investigation. |
| `monitor` | Take no enforcement action but increase observation of the affected scope. |

Because IITP responses are frequently coarse and high-impact, a Receiver SHOULD weight automated enforcement by the `severity` and `confidence` claims and SHOULD prefer reversible measures. Where an IITP event motivates per-subject enforcement, a Receiver that is also an SSF Transmitter SHOULD express that enforcement using the appropriate CAEP {{CAEP}} or RISC {{RISC}} events. Transmitters SHOULD send a follow-up event clearing the condition (`drift_status: reverted`, `attack_status: resolved`, or `trust_status: restored`) so that Receivers can lift any restrictions they applied.

# Privacy Considerations

IITP events describe environments, resources, and workloads rather than individual people, and therefore generally carry less personal data than CAEP or RISC events. Nonetheless, claims such as `indicators` (which may contain IP addresses) and `affected_principals` may relate to identifiable individuals. Transmitters SHOULD minimize such data to what the Receiver needs to act, and both parties MUST handle it in accordance with the Shared Signals Framework {{SSF}} and applicable law. Threat-actor attribution and indicators are sensitive and SHOULD only be shared with Receivers entitled to receive them.

# Security Considerations

Any implementation of the events described in this document SHOULD comply with the Shared Signals Framework {{SSF}}. Exchanging these events without complying with the Shared Signals Framework may result in security issues.

Because IITP events can motivate broad, environment-wide responses, they present a denial-of-service risk if abused: a malicious or spoofed Transmitter could induce a Receiver to freeze access to a healthy environment or to distrust a healthy workload. Receivers MUST authenticate the Transmitter and validate the Security Event Token in accordance with {{SSF}} and {{RFC8417}}, MUST only accept IITP events from Transmitters authorized for the affected environment, and SHOULD apply local policy -- including the `severity` and `confidence` claims and a preference for reversible measures -- before taking high-impact automated action. Receivers SHOULD have a path to rapidly reverse an action taken in response to an event later found to be erroneous, and SHOULD log IITP events and the actions taken for audit.

# IANA Considerations

This document has no IANA actions at this time. A future revision is expected to request registration of the `environment` subject identifier format in the Security Event Subject Identifier Formats registry, and of the IITP event type URIs.

--- back

# Use Cases
{:numbered="false"}

The following non-normative use cases motivated this profile and map to the event types defined above.

## Configuration Drift / Privilege Explosion
{:numbered="false"}

A detection provider such as a Cloud Infrastructure Entitlement Management product observes configuration drift in a cloud environment -- specifically a privilege explosion, where an IAM role (such as an S3 bucket administrator) is modified to include additional, powerful privileges (such as EKS privileges). The provider emits a `configuration-drift` event so the Identity Provider or other parties can act, for example by freezing all access except for administrators, or by restricting anyone holding the newly "poisoned" role until the environment is reverted to a known-good state.

## Application or Tenant Under Active Attack
{:numbered="false"}

An application or environment is actively under attack -- for example a password-spray campaign against a Salesforce tenant, or the detection of an adversary such as ShinyHunters within it -- a concern that does not fit neatly into enumerating every user account in existing risk models. The application instance emits an `asset-under-attack` event at the domain/tenant level so that other integrated applications (such as ServiceNow) can raise their overall detection sensitivity. The goal is to relay a detection to supporting components to minimize the blast radius.

## Supply Chain Compromise in a Homegrown Application
{:numbered="false"}

In an enterprise deployment, a supply-chain attack is detected in a homegrown application running in a Kubernetes cluster, resulting in a poisoned and actively compromised app. A `workload-compromise` event lets other systems know not to trust the compromised app, leading to protective measures such as blocking all OAuth and service-to-service traffic between service accounts, or preventing agents from communicating with the app.

# Acknowledgments
{:numbered="false"}

This draft profile is modeled on the structure and conventions of the OpenID Continuous Access Evaluation Profile 1.0 {{CAEP}} and the OpenID RISC Profile Specification 1.0 {{RISC}}, and is intended as input to discussion within the OpenID Foundation Shared Signals Working Group. The authors of those specifications and the members of the Working Group are gratefully acknowledged.

# Notices
{:numbered="false"}

This document is an independent working draft and is not a publication of the OpenID Foundation. It references OpenID Foundation specifications, which are subject to the OpenID Foundation's intellectual property and copyright policies. Any subsequent contribution of this material to the OpenID Foundation would be governed by those policies.

# Document History
{:numbered="false"}

-01

- Initial working group draft, converted from the 28 June 2026 HTML draft to kramdown-rfc source.
- Defines the `configuration-drift`, `asset-under-attack`, and `workload-compromise` event types.
- Defines the `environment` subject format and the `recommended_actions` registry.
- Additional candidate event types are tracked separately in the IITP Event Catalog and summarized in the IITP Event Reference {{IITP-EVENTS}}; they are not yet specified here.
