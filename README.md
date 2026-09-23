# OpenID Internet Infrastructure Threat Profile (IITP)

A proposed profile of the [Shared Signals Framework][ssf] for environment- and
application-level security signals.

Where [CAEP][caep] and [RISC][risc] describe the state of an individual subject
— a user, device, session, or account — IITP describes the state of the
*environment* those subjects operate in: cloud control planes, application
tenants, domains, and deployed workloads. A detection provider (a CIEM, SIEM,
or workload runtime security product) relays an environment-level detection to
Receivers positioned to attenuate access, raise detection posture, or withdraw
trust, in order to minimize the blast radius of a threat.

IITP events are **detections, not commands**. A Receiver retains full authority
over whether and how to respond.

## Documents

| Document | Source |
| --- | --- |
| OpenID Internet Infrastructure Threat Profile 1.0 — draft 01 | [`openid-iitp-1_0.md`](openid-iitp-1_0.md) |
| Proposed charter / work item | [`CHARTER.md`](CHARTER.md) |
| IITP Event Catalog — all candidate events | [`EVENT-CATALOG.md`](EVENT-CATALOG.md) |
| IITP Event Reference — per-event detail, claims, and examples | [`iitp-event-reference.html`](iitp-event-reference.html) |
| IITP / WISE profile comparison | [`openid-iitp-wise-comparison.html`](openid-iitp-wise-comparison.html) |

All five documents are published at <https://derrumbe.github.io/openid-iitp/>:

| Page | Link |
| --- | --- |
| Specification | <https://derrumbe.github.io/openid-iitp/> |
| Charter | <https://derrumbe.github.io/openid-iitp/charter.html> |
| Event catalog | <https://derrumbe.github.io/openid-iitp/event-catalog.html> |
| Event reference | <https://derrumbe.github.io/openid-iitp/iitp-event-reference.html> |
| IITP / WISE comparison | <https://derrumbe.github.io/openid-iitp/openid-iitp-wise-comparison.html> |

The specification is written in [kramdown-rfc][kr] markdown and rendered with
[xml2rfc][xr], the same toolchain used by the Shared Signals specifications.

## Related work

The [WISE profile][wise-repo] (Workload Identity Security Events) is a parallel
draft profile of the Shared Signals Framework, covering the workload identity
lifecycle — credential issuance and rotation, trust anchors, policy, and posture
— for workloads named by [WIMSE][wimse] or [SPIFFE][spiffe] identifiers.

IITP and WISE are complementary but do overlap: seven event pairs describe the
same underlying facts, and the two drafts take different positions on whether a
Transmitter is an authority or a detector. The
[IITP / WISE comparison](https://derrumbe.github.io/openid-iitp/openid-iitp-wise-comparison.html)
sets out where they collide and where each is unique.

| | Repository | Published draft |
| --- | --- | --- |
| WISE | <https://github.com/identitymonk/openid-wise> | <https://identitymonk.github.io/openid-wise/> |
| IITP | <https://github.com/derrumbe/openid-iitp> | <https://derrumbe.github.io/openid-iitp/> |

## Event types

Three event types are specified in draft 01:

| Event type | Signals |
| --- | --- |
| `configuration-drift` | Security configuration drifted from known-good — e.g. privilege explosion on a role |
| `asset-under-attack` | An app or tenant is under an active campaign |
| `workload-compromise` | A deployed workload is compromised or supply-chain poisoned |

Further candidate events are tracked in the IITP Event Catalog and are not yet
specified here. The
[IITP Event Reference](https://derrumbe.github.io/openid-iitp/iitp-event-reference.html)
details every event — specified and proposed — with its triggers, claims,
Receiver responses, and an example SET.

## Building

Requires `kramdown-rfc` (Ruby) and `xml2rfc` (Python):

```sh
gem install kramdown-rfc
pip install xml2rfc

make all       # -> build/openid-iitp-1_0.{html,txt}
make publish   # -> public/index.html
make clean
```

`make all` also produces a `.docx` when `pandoc` is available.

## Status

This is an independent working draft prepared as input to the OpenID Foundation
Shared Signals Working Group. **It is not a publication of the OpenID
Foundation** and confers no standards status. It references OpenID Foundation
specifications, which are subject to the Foundation's intellectual property and
copyright policies.

[ssf]: https://openid.net/specs/openid-sharedsignals-framework-1_0.html
[caep]: https://openid.net/specs/openid-caep-1_0.html
[risc]: https://openid.net/specs/openid-risc-1_0.html
[wise-repo]: https://github.com/identitymonk/openid-wise
[wimse]: https://datatracker.ietf.org/wg/wimse/about/
[spiffe]: https://spiffe.io/
[kr]: https://github.com/cabo/kramdown-rfc
[xr]: https://github.com/ietf-tools/xml2rfc
