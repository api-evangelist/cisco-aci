---
name: cisco-aci-triage-fabric-faults
description: Read the health of a Cisco ACI fabric and triage its faults — fabric health score, critical faults, node and interface state, and faults scoped to a specific tenant or EPG — through the APIC REST API or the CiscoDevNet ACI MCP server.
api: Cisco APIC REST API
generated: '2026-08-19'
method: generated
source: https://github.com/CiscoDevNet/mcp_server_cisco_aci_community/blob/main/scripts/server.py
operations:
  - aci_fabric_health_get
  - aci_critical_faults_get
  - aci_faults_get
  - aci_fabric_nodes_get
  - aci_fabric_links_get
  - aci_physical_interfaces_get
  - aci_ethernet_interfaces_get
  - aci_endpoints_get
  - aci_tenants_get
  - aci_endpoint_groups_get
---

# Triage faults and health in a Cisco ACI fabric

Read-only. Every operation here is a GET; none of them change the fabric. That makes this the right place to
start on an unfamiliar APIC.

## Get a session first

`POST https://{apic-hostname}/api/aaaLogin.json` with the `aaaUser` payload, then present the returned token
as the `APIC-cookie` cookie. Refresh with `aaaRefresh` inside `refreshTimeoutSeconds` (default 600) or reads
start returning **403**.

## Triage order

Work from the whole fabric inward. Answering "is anything wrong" before "what is wrong with this tenant"
saves you from chasing a tenant symptom that is really a dead uplink.

1. **Fabric health score** — `aci_fabric_health_get()`
   → `GET /api/node/class/fabricHealthTotal.json`. One number for the whole fabric.
2. **Critical faults only** — `aci_critical_faults_get()`
   → `GET /api/node/class/faultSummary.json` filtered to critical severity. Start here, not at the full
   fault list; a healthy fabric still carries hundreds of informational faults.
3. **Full fault summary** — `aci_faults_get()`
   → `GET /api/node/class/faultSummary.json`.
4. **Nodes and links** — `aci_fabric_nodes_get()` → `GET /api/node/class/fabricNode.json` (leaves, spines,
   APICs) and `aci_fabric_links_get()` → `GET /api/node/class/fabricLink.json`.
5. **Interfaces** — `aci_physical_interfaces_get()` → `GET /api/node/class/l1PhysIf.json` for configured
   state, `aci_ethernet_interfaces_get()` → `GET /api/node/class/ethpmPhysIf.json` for operational state.
   The pair matters: an interface can be admin-up and operationally down, and only the second class says so.
6. **Endpoints** — `aci_endpoints_get()` → `GET /api/node/class/fvCEp.json`. What the fabric has actually
   learned, as opposed to what policy says should be there.

## Scoping a fault query to one tenant

The class-wide calls above return the whole fabric. To ask about one tenant, query the managed object and
pull faults up through the subtree — this is the pattern Cisco uses in its own public Postman collection:

```
GET /api/mo/uni/tn-{tenant}.xml?query-target=subtree&target-subtree-class=fvAEPg&rsp-subtree=full&rsp-subtree-include=faults
```

Sort and page a large fault set rather than fetching it whole:

```
GET /api/class/faultInst.xml?rsp-prop-include=naming-only&order-by=faultInst.code|asc&page-size=100&page=0
```

## The 100,000-object wall

Unscoped class queries against a large fabric fail. Cisco documents it: the APIC does not respond with more
than 100,000 objects, and such queries "intermittently fail due to a timeout with error code 503" and the
message *"Unable to process the query, result dataset is too big"*. This is the single most common way a
naive fault sweep breaks. Do not retry the same unbounded query — narrow it:

- add `target-subtree-class` to name the class you actually want,
- add `query-target-filter`, e.g. `eq(faultInst.severity,"critical")`,
- add `rsp-prop-include=naming-only` to drop properties you will not read,
- page with `page-size` and `page`.

There are no rate-limit headers to guide you. Nothing tells you how close you are to the ceiling until the
503 arrives, so scope defensively rather than reactively.

## Resolving a fault code

A fault carries a numeric code, a severity, a description and the distinguished name of the object it is
attached to. The DN is the most useful field — it tells you exactly where in the tree the problem lives.
Cisco's published fault reference is an interactive lookup form
(https://www.cisco.com/c/dam/en/us/td/docs/Website/datacenter/syslogref/index.html), not a downloadable
registry, so an agent cannot resolve a code to its documented cause and remediation on its own. Surface the
code, the severity and the DN to a human and let them look it up.

## Watching instead of polling

Any of these queries can be turned into a live subscription by appending `?subscription=yes` and holding a
WebSocket (RFC 6455) to the APIC. Subscribing to `faultInst` gives a push stream of faults as they are
raised. There is no delivery guarantee and no replay: if the socket drops, re-query to resynchronise. See
`asyncapi/cisco-aci-event-subscriptions.yml`.

## References

- Error and fault semantics: `errors/cisco-aci-problem-types.yml`
- Query scoping and pagination: `conventions/cisco-aci-conventions.yml`
- Documented limits: `rate-limits/cisco-aci-rate-limits.yml`
- Tool → REST binding: `mcp/cisco-aci-tool-crosswalk.yml`
