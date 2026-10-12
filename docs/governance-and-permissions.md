# Governance and permissions in the open-source Kortix stack

Kortix is the open-source AI Operating System, and when you self-host it, governance decides what each person and each agent may do with the platform. Kortix governs at four levels: the account, the project, the resource (a connector, a secret, or an agent), and the individual connector call. All of it is configuration you can read in the repo or set in the app.

## Account roles and project roles

An account holds projects and the people who work in them. Each person holds exactly one account role and a project role on each project they work in.

| Account role | Can do |
| --- | --- |
| owner | Everything an admin can do, plus change the plan and payment method, delete the account, and manage owners. |
| admin | Rename the account, change its security settings, invite and remove members, manage groups, roles, and service-account keys, and create projects. |
| member | See the account and its plan, manage their own keys, and work only in the projects a grant gives them. |

A project has two roles: manager (every project permission) and member. Owners and admins are manager on every project in the account. Invite a teammate and give them access from the CLI:

```bash
kortix members invite alice@example.com --role member --project <project_id>:member
kortix access grant --user alice@example.com --agent kortix
kortix access grant --everyone --agent kortix
```

Without an agent grant, a member cannot start a session. Groups hold people, and a project role or a custom role can be bound to a group. An account role is bound to a person only; binding owner, admin, or member to a group returns 400.

## Per-resource permissions for people and agents

Kortix governs people and agents as principals. Per-resource permissions for people and agents, with roles, groups, and an audit trail, are part of the platform.

An agent's reach is declared in `kortix.yaml`. The behavior (prompt, model, mode, and the `permission` field) lives in `agents/<name>.md`; the governance block lives in the manifest. A real v2 block:

```yaml
kortix_version: 2
default_agent: kortix
agents:
  kortix:
    file: agents/kortix.md
    connectors: all
    secrets: all
    skills: all
    kortix_permissions: all
  reporter:
    file: agents/reporter.md
    connectors: [slack, docs-search]
    secrets: [STRIPE_API_KEY, GMAPS-primary]
```

The scoping field in the agent file is `permission`, not `tools`. A `kortix_permissions` list decides what the agent may do; a grant of `all` includes `project.gitops.merge`. To require a person to merge, give the agent a list without `project.gitops.merge`. Print every grantable entry with:

```bash
kortix validate --scopes
```

## Approval policies

A connector action runs, needs approval, or is blocked. The UI labels are Allow, Ask, and Block; the manifest names are `always_run`, `require_approval`, and `block`. A `require_approval` rule holds the call, and the agent's turn pauses and resumes when a person approves.

There is also a project-wide mode for connector actions:

```yaml
policy:
  default_mode: allow_all   # or risk
```

Approval gates are yours to set, and they are off until you set them. The default is `allow_all`. A `sensitive: true` connector changes the default for that connector: every action no rule matches needs approval, reads included.

## Server-side credentials

Two credential paths stay on your side of the wall.

Connector credentials are brokered server-side and never enter the machine. A connector spends a credential only for a caller in its audience; anyone else gets `credential_not_shared`, and no request leaves Kortix.

Secrets are encrypted at rest with a key per project, and no API, CLI command, or page returns a stored value. Each secret has an exposure. Environment is the default exposure; a session receives the value as an environment variable. Egress enforcement is a per-secret experimental option: the sandbox holds a handle, and Kortix substitutes the real value outside the sandbox, only on approved HTTPS hosts. An echoed value comes back as `[REDACTED]`. Use `--scope connector` to keep a value server-side for the connector gateway:

```bash
kortix secrets set STRIPE_API_KEY=sk_live_...
kortix secrets set PAYROLL_TOKEN=... --only-me
kortix secrets set ACME_API_KEY=... --scope connector
```

Every secret has a **Who can use it** audience: everyone in the project (the default), only you, or specific people, groups, or agents. A session receives a secret only when the audience of the value and the agent's `secrets` grant both allow it. Shared sessions, triggers, schedules, webhooks, and email sessions act for nobody, so they get only values shared with everyone. Secret values never appear in the repository.

## Reviewable work

Every session gets its own isolated sandbox and branch. Session work reaches main through a change request. Merge is default-deny for agents, so a person with the `project.gitops.merge` permission reads the diff before it lands. A session can merge its own change request only when its agent's `kortix_permissions` grant includes `project.gitops.merge`.

Every action is recorded. Reading, exporting, and streaming the audit log depend on the plan. On a self-hosted instance, an Enterprise entitlement unlocks SSO, SCIM, role-based access control, and audit; the SSO protocol is SAML 2.0 single sign-on, with SCIM 2.0 for provisioning.

Free to start, and free to self-host. [Get started](https://kortix.com).
