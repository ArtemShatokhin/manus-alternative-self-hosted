# Install and run the open-source Kortix stack

A self-hosted Kortix instance is the full open-source platform on a server you control: one Docker Compose stack that the `kortix self-host` CLI installs, configures, and updates. This page is for the operator who runs that server. Sessions run on a cloud sandbox provider, not on the server, so the instance must be reachable from the internet for those sandboxes to call back.

## Before you start

| Requirement | Why it matters |
| --- | --- |
| A Linux server with a public IP | The bootstrap installs Docker Engine and the Compose plugin. On a server that already has them, install only the CLI. |
| DNS A or AAAA records for `<domain>` and `api.<domain>` | Point both at the server before the first start. Caddy needs them to get the TLS certificate. |
| Ports 80 and 443 open inbound | Caddy terminates TLS and routes traffic on them. |
| A wildcard record `*.p.<domain>` | For sandbox previews. Use it on every public instance: without it, browser previews fall back to a path proxy that breaks root-absolute links, and `doctor` fails. |
| A sandbox provider key | Daytona, Platinum or E2B. Without one, sessions do not start. |
| A model provider key | You add it in the web app after the first sign-in. A self-hosted instance has no managed models. |

The CLI runs on Linux and macOS, on x86_64 and arm64.

## Install on a server that already has Docker

```bash
curl -fsSL https://kortix.com/install | bash
kortix self-host init --domain kortix.example.com --admin-email you@example.com
kortix self-host start
```

A first `init` in a terminal asks every setup question. `--yes`, or a shell with no terminal, skips the questions and keeps the defaults. `start` runs `init` first when the instance has no configuration. When the sandbox key is missing, `init` and `start` print a warning and continue.

`kortix self-host init` writes `docker-compose.yml`, `.env`, a `Caddyfile`, and `updater.sh` into `~/.config/kortix/self-host/<instance>/`. One Compose project, `kortix-<instance>`, holds every service.

## Install on a bare server

Kortix publishes a bootstrap script that installs Docker and the Compose plugin, installs the CLI, runs `kortix self-host init`, sets the domain, and runs `kortix self-host start`. Re-running it is safe. It runs on Linux only. The script is in the Kortix repository: [Kortix on GitHub](https://github.com/kortix-ai/suna).

When the script adds your user to the docker group, open a new shell before the next `kortix self-host` command.

## Evaluate on your own machine with a Cloudflare tunnel

To try Kortix with no domain and no DNS records:

```bash
kortix self-host init --tunnel cloudflare
kortix self-host start
```

The web app opens at `http://localhost:13737`. `cloudflared` publishes the API at an `https://<random>.trycloudflare.com` URL. That URL changes each time `cloudflared` restarts, and `start` and `update` capture the new one. For a stable URL, create a named tunnel in Cloudflare Zero Trust, point its public hostname at `http://kortix-api:8008`, then set both values:

```bash
kortix self-host env set CLOUDFLARE_TUNNEL_TOKEN=<token>
kortix self-host env set CLOUDFLARE_TUNNEL_HOSTNAME=kortix-api.example.com
kortix self-host start
```

## Reachability modes

| Mode | How you set it | Public URLs | Sessions run |
| --- | --- | --- | --- |
| domain | `--domain <domain>` | `https://<domain>` and `https://api.<domain>` | Yes |
| tunnel | `--tunnel cloudflare` | The app stays on `http://localhost:13737`; only the API is public, at the tunnel URL | Yes |
| local | neither flag | None: every port binds `127.0.0.1` | No, the sandbox cannot reach the API |

Domain mode is the production setup. Tunnel mode is for evaluation on your own machine. In local mode the services publish loopback ports: frontend 13737, API 13738, Supabase 13740. When the stack is stopped, `start` moves a port that is in use to a free one.

## Answer the setup questions

`kortix self-host configure` asks the setup questions again, in order. Each answer is pre-filled with the current value, so Enter keeps it. `configure` writes the files only. Run `kortix self-host start` to apply them.

| Step | What it sets |
| --- | --- |
| Reachability | domain (domain, API subdomain, ACME email) or tunnel (quick or named tunnel). |
| Admin email | `KORTIX_PLATFORM_ADMIN_EMAILS`, comma-separated. These people are platform admins. |
| Enterprise features | `ENTERPRISE_LICENSE_AVAILABLE`. With it enabled, it unlocks SSO, SCIM, role-based access control, and audit. |
| Restrict account creation | On by default. Only a platform admin can create a new account. |
| Sandbox provider | One of `daytona`, `platinum`, `e2b`, written to `ALLOWED_SANDBOX_PROVIDERS`. |

## First sign-in, and secure the instance

1. Open `https://kortix.example.com` and sign up with the admin email.
2. Connect GitHub at `/admin/git`, create the team account and a project, then add a model key.
3. Run `kortix self-host doctor` and fix every failing check.

Do steps 1 and 2 as soon as the stack starts. Until you set `EMAIL_URL` or `DISABLE_SIGNUP=true`, anyone who reaches the instance can create a confirmed account for any address, the admin address included. `kortix self-host configure` turns account-creation restriction on by default, so only a platform admin can create a new account after that; sign-ups, invites, SSO, and each person's personal account still work.

## What runs where

| Service | What it does |
| --- | --- |
| frontend | The web app. |
| kortix-api | The REST API at `/v1`. It relays any `/v1/llm` request to the gateway. |
| llm-gateway | The standalone LLM gateway. |
| kortix-migrate | A one-shot job that migrates the database before `kortix-api` starts. |
| kortix-updater | Pulls new images once a day, migrates, and rolls the app services. |
| supabase-* | The Supabase distribution, every image pinned by digest: Postgres, auth, REST, realtime, storage, and the rest. |
| caddy | Domain mode only. Terminates TLS on ports 80 and 443 and routes traffic. |
| cloudflared | Tunnel mode only. Publishes `kortix-api` through a Cloudflare tunnel. |

Off the box:

- **Sandboxes.** Every session runs on Daytona (the default), Platinum, or E2B. `kortix-api` creates them with your provider key, and the sandbox calls back to the instance's public API URL, `KORTIX_URL`.
- **Images.** `kortix/kortix-frontend`, `kortix/kortix-api`, and `kortix/kortix-gateway` on Docker Hub. Pulls need no credentials.
- **Model providers.** The gateway calls the providers your projects connect.

## Update

`kortix-updater` pulls new images once a day, migrates, and rolls the app services. It mounts the Docker socket.

Free to start, and free to self-host. [Get started](https://kortix.com).
