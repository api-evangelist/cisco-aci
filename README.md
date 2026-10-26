# Cisco ACI

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

Cisco Application Centric Infrastructure (ACI) is Cisco's data-center SDN fabric, programmed through the
Application Policy Infrastructure Controller (APIC) and its object model, the Management Information Tree
(MIT). Every APIC GUI, CLI and SDK action is executed through one REST interface: HTTPS requests to
`/api/mo/<dn>.json` and `/api/class/<class>.json` on the customer-operated controller, authenticated with a
cookie-based `aaaLogin` session token and refreshed with `aaaRefresh`. The API is model-driven rather than
OpenAPI-described.

## Ownership

Part of the Cisco family.

## Contract status

**No anonymously fetchable machine-readable contract.** Re-probed on 2026-08-19: `/openapi.json`,
`/openapi.yaml`, `/swagger.json`, `/v1/openapi.json`, `/api-docs` and every `/.well-known/*` path on
`developer.cisco.com` returned 404, and the DevNet always-on APIC sandbox host `sandboxapicdc.cisco.com`
returned 404 for `/openapi.json` and `/swagger.json` and 403 (auth required) for
`/api/class/fvTenant.json`. Cisco publishes an exhaustive object-model reference and a public Postman
workspace in place of a spec. API Evangelist has not authored a substitute.

## What Cisco does publish

- **APIC REST API Configuration Guide** — auth, query scoping, pagination, subscriptions, throttling.
- **Management Information Model Reference** — every MO class, property, fault and event type, per release.
- **A public Postman workspace** — "Cisco ACI - Public", 4 collections against `https://{{apic}}/api/...`.
- **A WebSocket (RFC 6455) event surface** — any query becomes a live subscription with `?subscription=yes`.
- **`security.txt` and `llms.txt`** on `www.cisco.com` — both real 200s, saved to this repo.
- **Infrastructure-as-code clients** — the `cisco.aci` Ansible collection and the `CiscoDevNet/aci`
  Terraform provider, both actively released.
- **A community MCP server** in the CiscoDevNet org — stdio only, 47 registered tools.
- **Product certifications** — Common Criteria EAL2+ for N9000/ACI mode with APIC 6.1(2g) and FIPS 140
  certificate 4747 for APIC/ACI Controller v6.1.

## Verified links

- [Developer portal](https://developer.cisco.com/site/aci/)
- [Documentation — APIC REST API Configuration Guide](https://www.cisco.com/c/en/us/td/docs/dcn/aci/apic/all/apic-rest-api-configuration-guide/cisco-apic-rest-api-configuration-guide-42x-and-later.html)
- [API reference — APIC Management Information Model Reference](https://developer.cisco.com/site/apic-mim-ref-api/)
- [Getting started](https://developer.cisco.com/docs/aci/getting-started/)
- [Postman — Cisco ACI Public workspace](https://www.postman.com/cisco-dcn-marketing-enablement/cisco-aci-public/overview)
- [Ansible collection](https://github.com/CiscoDevNet/ansible-aci)
- [Terraform provider](https://registry.terraform.io/providers/CiscoDevNet/aci/latest)
- [MCP server (community)](https://github.com/CiscoDevNet/mcp_server_cisco_aci_community)
- [DevNet always-on ACI sandbox](https://devnetsandbox.cisco.com/DevNet/catalog/aci-simulator-always-on)
- [security.txt](https://www.cisco.com/.well-known/security.txt)
- [Parent company](https://apis.io/providers/cisco/)

All URLs above returned HTTP 200 when probed on 2026-08-19.
