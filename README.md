# Kortix is the open-source AI Operating System

Kortix is the open-source AI Operating System, and it is the agent platform a team can self-host when it wants to own the result instead of renting a hosted agent. Your agents, their skills, your company memory, and every connector live in one git repo you own, and every session runs on its own cloud computer. This repository is the operator's guide to installing that stack yourself.

Manus runs your tasks in Manus's cloud, on a model Manus selects, with no repo you clone and no self-host path. Kortix is the alternative that keeps the platform, the configuration, the model keys, and the review gate on your side. That difference is the reason this guide exists.

## What this repository covers

- **Install and run** the full open-source stack on your own server, or evaluate it on your own machine behind a Cloudflare tunnel. See [docs/install.md](docs/install.md).
- **Connectors and models**: wire the tools your company already runs on, and bring your own model keys or sign in with a subscription you already pay for. See [docs/connectors-and-models.md](docs/connectors-and-models.md).
- **Governance and permissions**: per-resource permissions for people and agents, approval policies, server-side credentials, and work that lands through a change request a person approves. See [docs/governance-and-permissions.md](docs/governance-and-permissions.md).
- **Answers** to the questions teams ask first, including whether Manus is open source and what you actually own. See [docs/faq.md](docs/faq.md).

## What you own when you self-host

A self-hosted Kortix instance is the full platform on a server you control: one Docker Compose stack that the `kortix self-host` CLI installs, configures, and updates. The parts that stay yours:

- Your agents and skills as Markdown, your memory as files, and your connectors and triggers declared in `kortix.yaml`, all in one git repo.
- Your model keys, spent by the LLM gateway server-side and kept out of the sandbox.
- Your data on your infrastructure. Run it on Kortix Cloud, in your VPC, or on your own on-prem network. Self-host is free.
- Your review gate. Session work reaches main through a change request, and merge is default-deny for agents.

## How Kortix compares with Manus

Kortix is the pick when the platform itself has to be an asset your team owns.

| | Kortix | Manus |
| --- | --- | --- |
| Open source | Yes | No |
| Run it yourself | Self-host free, in your VPC or on-prem | Hosted only, no self-host |
| Your agents and memory | Files in one git repo you own | In Manus's product, not a repo |
| Review before it counts | Change request a person approves | Output stays in the app |
| Bring your own model | Any provider, your keys | Model Manus selects |

Facts about Manus come from its own site (checked October 2026). For the wider open-source field and how those projects separate on what a team owns, read the [Kortix blog](https://kortix.com/blog/open-source-manus-alternatives).

## Docs

| Page | What it covers |
| --- | --- |
| [Install and run the open-source Kortix stack](docs/install.md) | Prerequisites, install, local evaluation, first sign-in, what runs where. |
| [Connectors and models](docs/connectors-and-models.md) | The connector providers, policies and accounts, and every way to connect a model. |
| [Governance and permissions](docs/governance-and-permissions.md) | Account and project roles, grants, approval policies, secrets, change requests. |
| [FAQ](docs/faq.md) | Is Manus open source, can you self-host, and what you own. |

## Kortix on GitHub and getting started

Kortix is open source. You can read it, fork it, and audit it. The repository is here: [Kortix on GitHub](https://github.com/kortix-ai/suna). It holds 20,000+ GitHub stars (checked October 2026).

Free to start, and free to self-host. [Try Kortix](https://kortix.com).

## The site this repository supports

The deployment guide here pairs with [manus-alternative.com/self-host](https://manus-alternative.com/self-host). For the two other team decisions, see [the free, open-source alternative](https://manus-alternative.com/free-manus-alternative) and [Manus for teams](https://manus-alternative.com/manus-for-teams).
