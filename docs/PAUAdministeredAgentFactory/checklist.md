# Sky Core Review Checklist — `PAUAdministeredAgentFactory`

**Version:** 0.1.0 (draft) · **Last edited:** 2026-06-03

A reviewer checklist for validating a `PAUAdministeredAgentFactory` deployment and the arguments
passed to `deploy` / `deployFreezable`, before signing off on a Prime PAU deployment.

Modeled on the [Sky PE checklists](https://github.com/sky-ecosystem/pe-checklists). See the
[factory documentation](./README.md) for the full deploy flow and resulting permission layout, and the
[diamond-pau-deploy](https://github.com/sky-ecosystem/diamond-pau-deploy) repo for the underlying PAU
stack deployment.

> **Status:** living document. Items marked **(TBD)** await a definition from Soter / Sky Core
> (e.g. multisig `m/n` schemes, the `configurator` admin policy). Resolve them before first use.

## Conventions

- `* [ ]` items must each be verified and checked off.
- All addresses must be in **checksummed** form and cross-checked against the **chainlog** (or the
  agreed source of truth for this deployment).
- Role identifiers are `keccak256` of the role name — e.g. `ALLOCATOR_ROLE = keccak256("ALLOCATOR_ROLE")`,
  `ALLOCATOR_ADMIN_ROLE = keccak256("ALLOCATOR_ADMIN_ROLE")`, `DEFAULT_ADMIN_ROLE = 0x00`.

## Allowed configuration

This table is the **source of truth for what may appear in each deploy argument** for current
deployments. Edit it (and bump the revision) as policy evolves; the checklist below verifies a
deployment against this table rather than restating the policy inline.

> **Revision:** 0 (initial). **(TBD)** rows are not yet defined.

| Argument                              | Allowed value(s) for current deployments                                  |
| ------------------------------------- | ------------------------------------------------------------------------- |
| `admin`                               | One valid Sky subproxy (chainlog). Auto-granted admin on every component. |
| `integrationIds`                      | Approved, Beacon-registered facets only; may be empty.                    |
| `adminConfig.accessControlAdmins`     | Empty. **(TBD)**                                                          |
| `adminConfig.proxyAdmins`             | Empty. **(TBD)**                                                          |
| `adminConfig.rateLimitsAdmins`        | The `configurator` only, or empty. **(TBD: confirm policy)**             |
| `adminConfig.administeredAgentAdmins` | Empty. **(TBD)**                                                          |
| `roleAdminConfig`                     | Entries from the [Allowed role-admin assignments](#allowed-role-admin-assignments) table only. |
| `administeredAgentConfig.actors`      | Pre-vetted multisigs (`m/n` **TBD** by Soter).                           |
| `administeredAgentConfig.grantors`    | None (empty).                                                            |
| `administeredAgentConfig.revokers`    | Pre-vetted multisigs (`m/n` **TBD** by Soter).                          |
| `freezers` (`deployFreezable`)        | Pre-vetted multisigs (`m/n` **TBD**), same standard as `revokers`.       |

### Allowed role-admin assignments

Allowed `roleAdminConfig` entries. Each is applied as `AccessControls.setRoleAdmin(role, adminRole)`,
which sets the role *hierarchy* only — it does **not** grant either role to any address. Add rows here
as new roles are introduced.

| Role             | Administered by        | `role` (`bytes32`)                                                   | `adminRole` (`bytes32`)                                              |
| ---------------- | ---------------------- | ------------------------------------------------------------------- | ------------------------------------------------------------------- |
| `ALLOCATOR_ROLE` | `ALLOCATOR_ADMIN_ROLE` | `0x68bf109b95a5c15fb2bb99041323c27d15f8675e11bf7420a1cd6ad64c394f46` | `0x5dd5329f8a165b257688cafdac031f994a6462efe3eb1db3ef27aab8459f61d1` |

> `DEFAULT_ADMIN_ROLE` (`0x00`) must never appear as a `role` here — it is rejected on-chain by
> `CannotReassignDefaultAdminRole`. Holders of these admin roles (e.g. who actually holds
> `ALLOCATOR_ADMIN_ROLE`) are granted by separate transactions, **not** by this factory.

## Checklist

### 1. Factory & dependencies

* [ ] The `PAUAdministeredAgentFactory` source matches the audited commit, and the deployed bytecode
  matches that source.
* [ ] `pauFactory_` is the canonical, audited `PAUFactory` for this deployment (chainlog), and its
  `beacon()` is the intended Beacon.
* [ ] `administeredAgentFactory_` is the canonical, audited `AdministeredAgentFactory` (chainlog).

### 2. Deploy arguments

* [ ] Every argument matches the **[Allowed configuration](#allowed-configuration)** table.
* [ ] Every address is **checksummed** and verified against **chainlog** (the policy table says *which*
  address is allowed; this confirms the value is the *correct* one).
* [ ] `integrationIds` are registered on the factory's Beacon **before** this deploy, and each maps to
  the intended, audited facet (facet address + selector wiring reviewed).
* [ ] Every `roleAdminConfig` entry's `{role, adminRole}` pair — including the `bytes32` values —
  appears in the [Allowed role-admin assignments](#allowed-role-admin-assignments) table.
* [ ] IF freezable: the freezable variant is the intended choice for this Prime (the Controller is
  granted `ALLOCATOR_ROLE` on the proxy instead of `CONTROLLER`), and the freezer set is sufficient for
  the intended emergency-freeze response.

> **Enforced on-chain (informational — not review items).** The deploy reverts on these, so they cannot
> be true of a successful deployment: zero `admin` (`ZeroAdmin`); zero factory dependency
> (`ZeroPAUFactory` / `ZeroAdministeredAgentFactory`); duplicate or `admin`-overlapping agent entries
> (`AccountAlreadyActor` / `AccountAlreadyGrantor` / `AccountAlreadyRevoker` / `AccountAlreadyAdmin`); a
> `roleAdminConfig` entry targeting `DEFAULT_ADMIN_ROLE` (`CannotReassignDefaultAdminRole`).

### 3. Post-deploy

The factory wires every role and renounces its own deterministically — audited and covered by the test
suite — so given a correct factory (§1) and correct inputs (§2), the resulting permission layout follows
by construction. Per-role re-verification is therefore **not** required; the remaining checks are about
the deploy succeeding and its outputs being recorded correctly.

* [ ] The deploy transaction succeeded and emitted `PAUAdministeredAgentFactoryDeploy` with the
  expected addresses and configuration.
* [ ] The deployed addresses are recorded correctly in the deployment artifacts / chainlog.
* [ ] *(Optional, defense-in-depth)* Spot-check that the factory address holds no roles on the deployed
  contracts — redundant with §1 if the audited factory was used.

### 4. Sign-off

* [ ] All addresses double-checked against chainlog (or agreed source of truth).
* [ ] All **(TBD)** rows in the Allowed configuration table are resolved for this deployment.
* [ ] Reviewer 1: ____________________
* [ ] Reviewer 2: ____________________
