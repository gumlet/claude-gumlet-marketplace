# Gumlet MCP (OAuth)

Remote Gumlet MCP over OAuth — not API keys — for Claude Code, Claude.ai, Claude Desktop, Claude mobile, and Cowork.

MCP URL: `https://mcp.gumlet.com/mcp/v1`  
Scope: `gumlet.mcp`  
Auth metadata: [oauth-authorization-server](https://mcp.gumlet.com/.well-known/oauth-authorization-server)

## 1. Register these redirect URIs on the Gumlet OAuth app

Claude does not give you a per-app callback in a dashboard. Register these on the **same** Gumlet `client_id`.

`http://localhost:3118/callback` in [Authentication for connectors](https://claude.com/docs/connectors/building/authentication) is an **example**, not a required port. Claude Code binds a random loopback port each session (3118, 48201, …). Anthropic’s [Client ID Metadata Document](https://claude.ai/oauth/claude-code-client-metadata) declares `http://localhost/callback` and `http://127.0.0.1/callback`; the auth server should accept those with the **port ignored** (RFC 8252).

This plugin pins port **8080** (`oauth.callbackPort` / `--callback-port 8080`) so you can give Gumlet one exact URL if they cannot ignore the port.

| Surface | Redirect URI |
|---|---|
| Claude.ai, Desktop, mobile, Cowork | `https://claude.ai/api/mcp/auth_callback` |
| Claude Code (preferred: port-agnostic) | `http://localhost/callback` and `http://127.0.0.1/callback` (any port) |
| Claude Code (this plugin, exact match) | `http://localhost:8080/callback` and `http://127.0.0.1:8080/callback` |

Do not register `3118` unless you also change `callbackPort` to `3118`. That port is not special.

Do not use request-header API keys. Each user signs in with their own Gumlet account.

## 2. Claude Code (plugin)

From this marketplace directory:

```text
/plugin marketplace add ./gumlet-marketplace
/plugin install gumlet-plugin@gumlet-marketplace
```

Store the **app** client secret in the OS keychain (masked prompt). Do not put it in `.mcp.json`.

```bash
claude mcp add --transport http \
  --client-id 428c4418-4b29-488a-87b8-cdab674de611 \
  --client-secret \
  --callback-port 8080 \
  gumlet https://mcp.gumlet.com/mcp/v1
```

Then run `/mcp` and complete the Gumlet consent screen. After a GitHub publish, users add `YOUR_ORG/gumlet-marketplace` instead of the local path.

## 3. Claude.ai, Desktop, mobile, Cowork (custom connector)

These surfaces use Anthropic’s hosted callback, not localhost.

**Free / Pro / Max:** Customize → Connectors → Add custom connector  

**Team / Enterprise:** Organization settings → Connectors → Add → Custom (Web)

1. Remote MCP server URL: `https://mcp.gumlet.com/mcp/v1`
2. Authentication: **Always required** (OAuth, not “None” / request headers)
3. OAuth client: **Use your own OAuth client**
4. Client ID: `428c4418-4b29-488a-87b8-cdab674de611`
5. Client secret: the Gumlet app secret (Advanced settings)

Members click **Connect** on the connector and sign in at `https://webapp.gumlet.com/oauth/authorize`. Details: [Third party connectors with remote MCP](https://claude.com/docs/connectors/custom/remote-mcp).

## 4. So everyone can connect without pasting the app secret

Gumlet’s metadata requires `client_secret_basic` / `client_secret_post` and has no `registration_endpoint`, so Claude cannot use DCR or CIMD.

For a directory listing where users only click Connect:

1. Email `mcp-review@anthropic.com` and ask for `oauth_anthropic_creds` (Anthropic stores the client ID/secret and runs token exchange after user consent). Claude Code still uses its own local OAuth flow and does not use those stored creds.
2. Submit the plugin to the [community marketplace](https://platform.claude.com/plugins/submit) and, separately, apply for the Connectors Directory if you want it on claude.ai by default.

Until then, each Claude.ai org (or Claude Code user) enters the same Gumlet app `client_id` and secret once, then each person completes user OAuth.
