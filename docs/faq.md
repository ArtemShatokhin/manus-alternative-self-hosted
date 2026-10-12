# Open-source Kortix self-hosting FAQ

Kortix is the open-source AI Operating System, and this page answers the questions a team asks before it runs that platform on infrastructure it controls instead of renting a hosted agent.

## Is Manus open source?

No. Manus is a hosted, general-purpose AI agent that runs in Manus's own cloud and is not open source. There is no repo you clone, no self-host path, and no way to swap the model: Manus selects it. The browser operator installs as a Chrome extension, and the work still runs through Manus. Manus sells on credits, with a Team tier that adds a shared credit pool, an admin dashboard, SSO, and audit logs.

Manus is a strong product for a person or a small team that wants a capable hosted agent and no setup. The trade-off is ownership: your configuration, your model choice, your review process, and the infrastructure all sit on Manus's side. For the wider field and how the open-source projects separate on what a team owns, read the [Kortix blog](https://kortix.com/blog/open-source-manus-alternatives).

## Can you self-host an AI agent platform like Manus?

Yes. Kortix is the open-source AI Operating System, and it is the platform a team can self-host. A self-hosted instance is the full platform on a server you control: one Docker Compose stack that the `kortix self-host` CLI installs, configures, and updates. Run it on Kortix Cloud, in your VPC, or on your own on-prem network. Self-host is free.

Two things the instance still needs, and both are normal for a self-hosted agent platform:

- **A cloud sandbox provider key.** Sessions run on Daytona (the default), Platinum, or E2B, not on the server itself. Without a provider key, sessions do not start.
- **A model key.** A self-hosted instance has no managed models, so you connect your own providers, or the ChatGPT plan you already pay for, or sign in with OpenCode Zen and Go.

The instance also has to be reachable from the internet, because the sandbox calls back to its public API URL. The install steps and the prerequisites are on [the install page](install.md).

## What do you actually own when you self-host?

You own the system, not just the output. Your agents and skills are Markdown, your company memory is files, and your connectors and triggers are declared in `kortix.yaml`. All of it sits in one git repo, so a person reads it and an agent edits it, and you can `grep` the whole company. Around that repo you own the model keys, the credential broker, the permission model, the audit trail, and the review gate.

Here is the same set of questions answered for both platforms.

| | Kortix | Manus |
| --- | --- | --- |
| Open source | Yes | No |
| Run it yourself | Self-host free, in your VPC or on-prem | Hosted only, no self-host |
| Your agents and memory | Files in one git repo you own | In Manus's product, not a repo |
| Review before it counts | Change request a person approves | Output stays in the app |
| Bring your own model | Any provider, your keys | Model Manus selects |

The four things a team cannot keep on a hosted product are the four things Kortix keeps on your side: the configuration as files you can diff and roll back, the model choice, a review gate, and the infrastructure. When the work belongs to a team instead of one person, those are the parts that decide the platform.

Free to start, and free to self-host. [Try Kortix](https://kortix.com).
