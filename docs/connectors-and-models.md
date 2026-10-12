# Connectors and models in the open-source Kortix stack

Kortix is the open-source AI Operating System, and on a self-hosted instance the connectors and the model keys are yours. Two things turn a bare self-hosted instance into a working agent platform: connectors, so agents reach the tools your company already runs on, and model credentials, so every session has a model to run on. Both stay on your infrastructure.

## Connectors

A connector gives a project governed access to one external app, MCP server, or API. Agents call it as a tool, and your own code calls it through the SDK. Kortix stores the credential, picks the account, applies the project's policies, calls the provider, and writes an audit event for every call. Connector credentials are brokered server-side and never enter the machine.

Kortix connects 3,000+ apps in a click, plus MCP, OpenAPI, Postman, GraphQL, and raw HTTP.

| Provider | Use it for | Required fields |
| --- | --- | --- |
| composio | The managed catalog of SaaS apps (Gmail, GitHub, Linear). The default for managed apps. | app |
| mcp | A remote MCP server. | url; transport is http or sse |
| openapi | An API with an OpenAPI document. | spec |
| postman | An API described by a Postman collection. | spec |
| graphql | A GraphQL endpoint. Kortix introspects it when `spec` is absent. | endpoint |
| http | A raw HTTP API. | base_url |
| channel | Slack, Microsoft Teams, or email. Kortix writes it when you connect a channel. | platform |
| computer | Built in. Each connected computer is one account. Never written to `kortix.yaml`. | none |

A connector lives in `kortix.yaml` under `connectors:`, or in a file it lists under `imports:`. The manifest never holds a credential. A real block:

```yaml
connectors:
  - slug: gmail
    name: Gmail
    provider: composio
    app: gmail
  - slug: docs-search
    provider: mcp
    url: https://mcp.example.com/mcp
  - slug: billing-api
    name: Billing API
    provider: openapi
    spec: https://api.example.com/openapi.json
    auth:
      type: bearer
    policies:
      - match: "get_*"
        action: always_run
      - match: "*"
        action: require_approval
```

### Add a connector

From the web app: open **Customize → Connectors**, find the app on the **All** tab, and select **Install**. Kortix adds the connector and opens its page, where you select **Add account** to authorize one.

From the CLI:

```bash
kortix connectors apps gmail
kortix connectors add gmail --provider composio --app gmail --apply
kortix connectors connect gmail
```

Without `--apply`, `add` and `rm` edit your local `kortix.yaml`. Run `kortix ship` to apply the change, then `kortix connectors sync` to reconcile the connector. With `--apply`, Kortix commits the change to the default branch and syncs at once, like the web app.

### Policies and accounts

Each connector action is `always_run`, `require_approval`, or `block`, shown in the UI as Allow, Ask, or Block. A `require_approval` rule holds the call, and the agent's turn pauses and resumes when a person approves. An action rule matches down to the arguments of each call.

A connector holds one or more accounts, and each account is the authorized identity a call runs as. A shared account uses `owner_type: project` and is available to everyone in the project, and to every agent whose `connectors` grant lists the connector, triggers included. A private account uses `owner_type: member` and is available only to its owner in the owner's private sessions. An unattended run never uses a private account.

## Bring your own model keys

On Kortix Cloud a session runs on the platform default. A self-hosted instance has no managed models, so you connect your own providers. You can use any model provider with your own keys, or the ChatGPT plan you already pay for, or sign in with your OpenCode Console account for OpenCode Zen and Go.

Connect a provider from the web app under **Customize → Models → Providers**, or from the CLI on the linked project:

```bash
kortix providers set anthropic          # reads the key from stdin
kortix providers set openai sk-...      # key as an argument
kortix providers set bedrock --region us-east-1
kortix providers ls                     # what is connected
kortix providers rm anthropic           # remove the key and any login
```

`kortix providers set` stores the key as an encrypted project secret under the provider's key name. With the LLM gateway on, Kortix spends those keys server-side and keeps them out of the sandbox. The secret names are `ANTHROPIC_API_KEY` (anthropic), `OPENAI_API_KEY` (openai), `OPENROUTER_API_KEY` (openrouter), `GOOGLE_GENERATIVE_AI_API_KEY` (google), `GROQ_API_KEY` (groq), `XAI_API_KEY` (xai), `DEEPSEEK_API_KEY` (deepseek), `MISTRAL_API_KEY` (mistral), `OPENCODE_API_KEY` (OpenCode Zen), `OPENCODE_GO_API_KEY` (OpenCode Go), and `AWS_BEARER_TOKEN_BEDROCK` plus `AWS_REGION` (Amazon Bedrock). Other providers connect in the Providers form with the key name the form shows.

### Sign in instead of a key

Three providers take a device-code login:

```bash
kortix providers login openai        # ChatGPT Plus or Pro
kortix providers login opencode      # OpenCode Zen
kortix providers login opencode-go   # OpenCode Go
```

An OpenCode login lasts 30 days, and Kortix renews it one day before it expires. When a renewal is refused, requests return `provider_reauth_required`: sign in again.

### A custom OpenAI-compatible endpoint

For a self-hosted or unlisted OpenAI-compatible endpoint, open the **Custom** tab. Enter a provider id, a name, the base URL, a model id, and a model name. The key is optional. Kortix saves the key as `CUSTOM_<ID>_API_KEY` and shows an OpenCode provider snippet. Commit the snippet to `opencode.jsonc` in the project's OpenCode config directory (`harnesses/opencode/` by default). A custom provider calls its endpoint directly, not through the gateway, so budgets, logs, and routing do not apply to it.

### Pick a model

A model id has one of three shapes. A self-hosted instance uses the second and third.

| Shape | Example | Who supplies the credentials | Who bills |
| --- | --- | --- | --- |
| Bare managed id | kimi-k3 | Kortix | Kortix credits |
| provider/model | anthropic/your-model-id | Your provider API key | Your provider |
| codex/id | codex/your-model-id | Your ChatGPT Plus or Pro subscription | OpenAI, through your plan |

Choose a model per agent, per session, or per message. From the CLI:

```bash
kortix models default <model-id>          # project default
kortix models default <model-id> --account # account default
kortix agents model <agent> <model-id>     # pin one agent
kortix sessions new --model <provider>/<model-id>
```

The gateway finds the credentials, applies the project's routing, access rules, and budgets, and records the cost. Kortix charges nothing for a request that runs on your own key or subscription; your provider bills you.

Free to start, and free to self-host. [Try Kortix](https://kortix.com).
