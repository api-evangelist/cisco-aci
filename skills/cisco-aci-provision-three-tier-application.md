---
name: cisco-aci-provision-three-tier-application
description: Provision a three-tier (web/app/db) application in a Cisco ACI tenant — VRF, bridge domains, application profile, EPGs, filters and contracts — through the APIC REST API or the CiscoDevNet ACI MCP server.
api: Cisco APIC REST API
generated: '2026-08-19'
method: generated
source: https://github.com/CiscoDevNet/mcp_server_cisco_aci_community/blob/main/scripts/server.py
operations:
  - aci_tenant_create
  - aci_vrf_create
  - aci_bridge_domain_create
  - aci_application_profile_create
  - aci_epg_create
  - aci_filter_create
  - aci_filter_entry_create
  - aci_contract_create
  - aci_contract_subject_create
  - aci_epg_contract_provider_bind
  - aci_epg_contract_consumer_bind
  - aci_epg_domain_bind
  - aci_subnet_create
  - aci_create_3tier_app
---

# Provision a three-tier application in Cisco ACI

**Before anything else, read this.** These operations write to a production data-center fabric. There is no
test mode and no dry run: the same credentials, the same endpoints and the same object model apply to a lab
APIC and to the fabric carrying live traffic. The only thing separating them is the hostname you point at.
Confirm which APIC you are addressing before the first write.

## Preconditions

- A reachable APIC and an account with write privileges in the target security domain.
- A session: `POST https://{apic-hostname}/api/aaaLogin.json` with
  `{"aaaUser": {"attributes": {"name": "<user>", "pwd": "<password>"}}}`. Keep the returned token as the
  `APIC-cookie` cookie and refresh it with `aaaRefresh` before `refreshTimeoutSeconds` (default 600) elapses,
  or every subsequent call returns **403**.
- Know the physical or VMM domain name you will bind the EPGs to. Without a domain binding the EPGs exist in
  policy but nothing lands on them.

## Order of operations

ACI is declarative but not order-free: a bridge domain needs its VRF, an EPG needs its bridge domain and its
application profile, and a contract binding needs both EPGs. Build outward from the tenant.

1. **Tenant** — `aci_tenant_create(tenant_name, description?)`
   → `POST /api/node/mo/uni.json`
2. **VRF** — `aci_vrf_create(tenant_name, vrf_name, description?)`
   → `POST /api/node/mo/uni/tn-{tenant_name}.json`
3. **Bridge domains, one per tier** — `aci_bridge_domain_create(tenant_name, vrf_name, bd_name, subnet_ip?)`
   → `POST /api/node/mo/uni/tn-{tenant_name}.json`.
   Add further gateways with `aci_subnet_create(tenant_name, bd_name, subnet_ip, scope?)`
   → `POST /api/node/mo/uni/tn-{tenant_name}/BD-{bd_name}.json`. `scope` defaults to `public`; use `private`
   unless you intend the subnet to be advertised out of the fabric.
4. **Application profile** — `aci_application_profile_create(tenant_name, app_name, description?)`
5. **EPGs, one per tier** — `aci_epg_create(tenant_name, app_name, epg_name, bd_name)`
   → `POST /api/node/mo/uni/tn-{tenant_name}/ap-{app_name}.json`
6. **Domain binding** — `aci_epg_domain_bind(tenant_name, app_name, epg_name, domain_name, domain_type?)`
   → `POST /api/node/mo/uni/tn-{tenant_name}/ap-{app_name}/epg-{epg_name}.json`.
   `domain_type` defaults to `phys`.
7. **Filters** — `aci_filter_create(tenant_name, filter_name)` then
   `aci_filter_entry_create(tenant_name, filter_name, entry_name, protocol?, dst_port?)`
   → `POST /api/node/mo/uni/tn-{tenant_name}/flt-{filter_name}.json`
8. **Contracts** — `aci_contract_create(tenant_name, contract_name)` then
   `aci_contract_subject_create(tenant_name, contract_name, subject_name, filter_name)`
   → `POST /api/node/mo/uni/tn-{tenant_name}/brc-{contract_name}.json`
9. **Bind the contract to both sides** — the tier that offers the service is the *provider*, the tier that
   calls it is the *consumer*:
   - `aci_epg_contract_provider_bind(tenant_name, app_name, epg_name, contract_name)`
   - `aci_epg_contract_consumer_bind(tenant_name, app_name, epg_name, contract_name)`

   Bind only one direction and traffic is silently denied. ACI is default-deny between EPGs; a missing
   consumer binding does not error, it just drops.

## The shortcut, and when not to take it

`aci_create_3tier_app(tenant_name, app_name, vrf_name?)` and
`aci_create_web_app_stack(tenant_name, app_name, web_subnet?, app_subnet?)` fan out to the steps above in one
call. They are convenient and they are also the riskiest tools in the server: a single tool call writes
dozens of managed objects with default names and default subnets. Use them for a lab or a demo. For anything
that matters, run the numbered steps so each write is separately reviewable.

## Retries and failure

- **POST is idempotent.** Cisco documents it: replaying the same POST with the same input has no additional
  effect, because the payload declares the desired state of a managed object at a distinguished name. A
  retried step converges rather than duplicating.
- **But the sequence is not transactional.** There is no Idempotency-Key, no request grouping and no
  rollback. If step 7 fails, steps 1–6 stay applied. Re-run from the failed step; do not tear down and start
  over unless you intend to delete.
- **403 usually means the session expired,** not that you lack permission. Re-run `aaaLogin` and replay.
- **Verify before declaring success.** Read back with `aci_endpoint_groups_get()` and
  `aci_contracts_get()`, or scope a query directly:
  `GET /api/mo/uni/tn-{tenant}.json?query-target=subtree&target-subtree-class=fvAEPg&rsp-subtree=full&rsp-prop-include=config-only`.
  A 200 with an empty `imdata` array is the normal response to a successful write — it is not evidence that
  the object exists.

## Deletion

`aci_tenant_delete(tenant_name)` removes the tenant **and everything inside it** — every application profile,
EPG, bridge domain, contract and subnet. It is a single call with no confirmation step. Treat it as
irreversible.

## References

- Auth, sessions and the challenge token: `authentication/cisco-aci-authentication.yml`
- Query scoping, pagination, idempotency: `conventions/cisco-aci-conventions.yml`
- Object model and relationships: `data-model/cisco-aci-data-model.yml`
- Tool → REST binding for all 47 tools: `mcp/cisco-aci-tool-crosswalk.yml`
