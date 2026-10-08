# OAuth redirect URIs for Securin IAM

Audience: Securin identity and platform administrators who manage the Keycloak realm `CyberSecurityWorks`.

This plugin repository does not ship tokens, client secrets, or user credentials. Every store install uses OAuth. Each person signs in as themselves, including on a shared team bot. A bearer token on `https://mcp.securin.io/api/mcp` is one shared identity and is not for Cursor Marketplace or Claude directory users.

## What to change

Client that Dynamic Client Registration returns today: `securin-java-ai-mcp`.

The authorization server advertises `registration_endpoint` `https://mcp.securin.io/register`. That endpoint currently returns the fixed client id above rather than a new client per installation. Until registration can issue a distinct client per vendor, add the redirect URIs to `securin-java-ai-mcp`.

A dedicated client per vendor (one for Cursor and Grok Bot, one for Claude) is cleaner, but only after `https://mcp.securin.io/register` returns that vendor client. If registration keeps returning `securin-java-ai-mcp`, extra clients are never used by the plugins in this repository.

Realm: `CyberSecurityWorks`  
Issuer: `https://auth.securin.io/auth/realms/CyberSecurityWorks`  
Authorization endpoint: `https://auth.securin.io/auth/realms/CyberSecurityWorks/protocol/openid-connect/auth`  
Token endpoint used by MCP clients: `https://mcp.securin.io/token`  
MCP resource: `https://mcp.securin.io/mcp`  
Protected resource metadata: `https://mcp.securin.io/.well-known/oauth-protected-resource`  
Authorization server metadata: `https://mcp.securin.io/.well-known/oauth-authorization-server`

Do not point plugins at `https://mcp.securin.io/api/mcp`. That URL is the bearer-token endpoint.

## Redirect URIs by client

Probes on 8 October 2026 called the authorize endpoint with `client_id=securin-java-ai-mcp` and did not call `POST /register`. A result of login page means HTTP 200 and the page title `Log in to Securin`, with no `Invalid parameter: redirect_uri` error. A result of rejected means HTTP 400 and that error, so the user never sees the login form.

### Cursor and Grok Bot

Cursor documents two fixed callbacks. The same pair is what current Cursor and Grok Bot builds send. Cloud agents and Grok Bot use the https callback. The desktop app uses the loopback callback. Some desktop installs and Dynamic Client Registration payloads still include the `cursor://` scheme. Grok Bot no longer sends `grokbot://` (Cursor staff, 7 September 2026).

| Redirect URI | Surface | Probe on 8 October 2026 | Action |
| --- | --- | --- | --- |
| `https://www.cursor.com/agents/mcp/oauth/callback` | Cursor Agents, Grok Bot cloud, web | Rejected | Add. This is the known failure: Keycloak returns `Invalid parameter: redirect_uri` and the connector stays unauthenticated. |
| `http://localhost:8787/callback` | Cursor and Grok Bot desktop | Login page | Already allowed. Keep it. |
| `http://127.0.0.1:8787/callback` | Desktop loopback variant | Login page | Already allowed under the localhost wildcard described below. Keep it. |
| `https://cursor.com/agents/mcp/oauth/callback` | Same path without `www`. Not in the published Cursor pair. | Rejected | Add as defense. A client that drops `www` fails the same way today. |
| `cursor://anysphere.cursor-mcp/oauth/callback` | Older desktop installs, and still one entry in the DCR `redirect_uris` array | Rejected | Add if this client is allowed to store a custom scheme. Registration that submits this URI together with the https callback can fail the whole registration on providers that reject non-http schemes. |

Do not add `grokbot://mcp/oauth/callback`. It is rejected today and current Grok Bot builds do not send it.

Sources: [Cursor MCP static redirect URLs](https://cursor.com/docs/mcp), Cursor forum confirmation that DCR sends the `cursor://` URI, the `https://www.cursor.com/agents/mcp/oauth/callback` URI, and `http://localhost:8787/callback`.

### Claude.ai, Claude Desktop, Claude mobile, and Cowork

Anthropic documents one hosted callback for claude.ai, Claude Desktop, Claude mobile, and Cowork. There is no `claude.com` callback in the current authentication guide.

| Redirect URI | Surface | Probe on 8 October 2026 | Action |
| --- | --- | --- | --- |
| `https://claude.ai/api/mcp/auth_callback` | Hosted Claude apps | Login page | Already allowed. Keep this exact value. No trailing slash. |
| `https://www.claude.ai/api/mcp/auth_callback` | Not documented | Rejected | Do not add unless a capture shows Claude sending it. |
| `https://claude.com/api/mcp/auth_callback` | Not documented | Rejected | Do not add unless a capture shows Claude sending it. |
| `https://www.claude.com/api/mcp/auth_callback` | Not documented | Rejected | Do not add unless a capture shows Claude sending it. |

Source: [Authentication for connectors](https://claude.com/docs/connectors/building/authentication). The guide says to register exactly `https://claude.ai/api/mcp/auth_callback`.

### Claude Code

Claude Code is a native client. It uses an RFC 8252 loopback redirect on an ephemeral port, for example `http://localhost:3118/callback`. Its Client ID Metadata Document declares `http://localhost/callback` and `http://127.0.0.1/callback` and expects the authorization server to ignore the port. Authorization server metadata here does not advertise Client ID Metadata Documents (`client_id_metadata_document_supported` is absent, and `token_endpoint_auth_methods_supported` does not include `none`), so Claude falls back to Dynamic Client Registration and the fixed client above.

| Redirect URI | Probe on 8 October 2026 | Action |
| --- | --- | --- |
| `http://localhost:<port>/callback` | Login page for ports 9, 3118, 8787, and 65535 | Already covered if the wildcard below stays. |
| `http://127.0.0.1:<port>/callback` | Login page for ports 1, 3118, 8787, and 65535 | Already covered if the wildcard below stays. |
| `http://localhost/callback` | Rejected | Add. This is the portless form in the Claude Code client metadata. Live sessions include a port, which already works. |
| `http://127.0.0.1/callback` | Rejected | Add, for the same reason. |

`https://localhost:8787/callback` and `http://[::1]:8787/callback` are rejected. Current Claude Code builds use `http` with `localhost` or `127.0.0.1`, not those forms.

### mcp-remote (existing Claude Desktop bridge)

`npx mcp-remote` and the Claude Desktop `.mcpb` bundle still use a local callback. Current mcp-remote (0.14.3) listens on `http://localhost:<port>/oauth/callback`. The port is `3335 + (int(md5(serverUrl)[:4], 16) % 45816)`, then the next free port among 8 candidates. For `https://mcp.securin.io/mcp` that base port is **38637**, so the candidates are **38637 through 38644**.

| Redirect URI | Probe on 8 October 2026 | Action |
| --- | --- | --- |
| `http://localhost:38637/oauth/callback` | Login page | Already allowed. Keep. |
| `http://localhost:38638/oauth/callback` and `http://localhost:38644/oauth/callback` | Login page | Already allowed. Keep the whole candidate range, or rely on the wildcard. |
| `http://127.0.0.1:38637/oauth/callback` | Login page | Already allowed. Keep, for `--host 127.0.0.1`. |
| `http://localhost:3334/oauth/callback` | Login page | Older default port. Already allowed. No need to add a special entry if the wildcard stays. |

The plugin `.mcp.json` for Claude Code now uses remote HTTP directly, so new Claude Code installs do not need mcp-remote. The `.mcpb` package and the example in `docs/examples/claude-mcp-remote.json` still do.

## Localhost wildcard already in effect

These authorize checks succeeded even when the path was not a known callback (`/`, `/evil`) and when the port was 1, 9, or 65535:

- `http://localhost:<port>/<path>`
- `http://127.0.0.1:<port>/<path>`

The same hosts with no port were rejected. That pattern matches a Keycloak valid-redirect wildcard of the form `http://localhost:*` and `http://127.0.0.1:*` (any port, any path). This document did not read the Keycloak admin screen, so confirm the saved patterns there.

Do not tighten that wildcard in the same change that adds the Cursor https callback. mcp-remote uses `/oauth/callback`, Claude Code and Cursor desktop use `/callback`, and a tighter rule that keeps only one path will break the other. If you tighten later, keep at least:

- `http://localhost:*/callback`
- `http://127.0.0.1:*/callback`
- `http://localhost:*/oauth/callback`
- `http://127.0.0.1:*/oauth/callback`

A wildcard that accepts every path on localhost is broad. Any local process can bind a port and receive the authorization code. The MCP authorization spec already warns about loopback redirects. Prefer the four patterns above when you next edit this client, after you confirm no other integrator depends on a different path.

## Minimum list to add

Add these to Valid Redirect URIs on `securin-java-ai-mcp`. Items marked keep are already accepted and must stay.

Add now:

1. `https://www.cursor.com/agents/mcp/oauth/callback`
2. `https://cursor.com/agents/mcp/oauth/callback`
3. `cursor://anysphere.cursor-mcp/oauth/callback`
4. `http://localhost/callback`
5. `http://127.0.0.1/callback`

Keep (already accepted on 8 October 2026):

1. `https://claude.ai/api/mcp/auth_callback`
2. `http://localhost:8787/callback`
3. `http://127.0.0.1:8787/callback`
4. `http://localhost:*/callback` and `http://127.0.0.1:*/callback` (Claude Code ephemeral ports)
5. `http://localhost:38637/oauth/callback` through `http://localhost:38644/oauth/callback`, or the existing localhost port wildcard that already covers them

## Web origins

Valid Redirect URIs are the setting that fixes `Invalid parameter: redirect_uri`. Web Origins are a separate Keycloak CORS list.

Cursor, Claude, and mcp-remote exchange the authorization code from their own process, not from browser JavaScript on cursor.com or claude.ai. The token URL they call is `https://mcp.securin.io/token`. A missing web origin does not cause the redirect error this probe recorded.

If the login page appears and the browser then fails with a CORS error, add these Web Origins and nothing broader:

- `https://www.cursor.com`
- `https://cursor.com`
- `https://claude.ai`
- `http://localhost`
- `http://127.0.0.1`

Do not set Web Origins to `*`.

## Per-user OAuth, not a shared bearer token

Each user, including each person on a shared or team bot, signs in to Securin and receives tokens for their own account. The MCP server enforces that account's workspaces and permissions.

`https://mcp.securin.io/api/mcp` with a bearer token is a different mode. One token is one application identity shared by every caller. Do not put that token in a marketplace plugin, and do not tell directory users to paste it.

## How to verify

After saving the client, run one request per redirect URI. A login page is success. `Invalid parameter: redirect_uri` is failure. You do not need to finish sign-in.

```bash
CHALLENGE="$(python3 - << 'PY'
import base64, hashlib
verifier = "probe-verifier-securin-oauth-redirect-check-2026"
print(base64.urlsafe_b64encode(hashlib.sha256(verifier.encode()).digest()).decode().rstrip("="))
PY
)"

redirect_uri='https://www.cursor.com/agents/mcp/oauth/callback'
curl -sS -D - -o /tmp/securin-auth-body.html \
  --get "https://auth.securin.io/auth/realms/CyberSecurityWorks/protocol/openid-connect/auth" \
  --data-urlencode "response_type=code" \
  --data-urlencode "client_id=securin-java-ai-mcp" \
  --data-urlencode "redirect_uri=${redirect_uri}" \
  --data-urlencode "scope=openid" \
  --data-urlencode "state=probe" \
  --data-urlencode "code_challenge=${CHALLENGE}" \
  --data-urlencode "code_challenge_method=S256"

echo "---- body signals ----"
grep -E "Invalid parameter: redirect_uri|Log in to Securin" /tmp/securin-auth-body.html || true
```

Repeat with each URI in the tables. Success is HTTP 200, title `Log in to Securin`, and no `Invalid parameter: redirect_uri` line. Failure is HTTP 400 with that line.

Discovery check:

```bash
curl -sS https://mcp.securin.io/.well-known/oauth-authorization-server | python3 -m json.tool
curl -sS https://mcp.securin.io/.well-known/oauth-protected-resource | python3 -m json.tool
```

Expect `registration_endpoint` `https://mcp.securin.io/register`, `code_challenge_methods_supported` including `S256`, and protected resource `https://mcp.securin.io/mcp` with scope `openid`.

## Probe notes that are not redirect URIs

Recorded from public metadata on 8 October 2026:

- `authorization_servers` on the protected resource document is `https://mcp.securin.io` (path form uses `https://mcp.securin.io/`). Claude uses only the first entry. Metadata for that issuer is served on `mcp.securin.io`, and the authorization endpoint inside it is on `auth.securin.io`.
- Unauthenticated `POST https://mcp.securin.io/mcp` returns 401 with `WWW-Authenticate` pointing at `https://mcp.securin.io/.well-known/oauth-protected-resource/mcp`.
- `token_endpoint_auth_methods_supported` lists confidential methods only (`client_secret_basic`, `client_secret_post`, `private_key_jwt`, `client_secret_jwt`, `tls_client_auth`). It does not list `none`. Claude public clients and Client ID Metadata Documents expect `none`. Existing mcp-remote sign-in still works, so the fixed client accepts a public token exchange in practice. Confirm that `https://mcp.securin.io/token` keeps accepting that client without a secret after the redirect change. Do not put a client secret in this repository.
- This probe did not `POST` to `/register`, so it did not change the registered client.
