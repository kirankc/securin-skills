# Submission checklist: Cursor Marketplace and Claude directory

One plugin, `securin-platform` at version `0.1.0`, is packaged for both stores. Skills stay in `skills/`. Neither marketplace gets a second copy.

Do not submit until Kiran has reviewed the pull request. Do not mark that pull request ready for review, and do not merge it, as part of packaging work.

## Shared facts

| Item | Value |
| --- | --- |
| Plugin name | `securin-platform` |
| Version | `0.1.0` |
| Publisher | Securin Inc |
| Homepage | `https://www.securin.io` |
| Repository | `https://github.com/securin-public/securin-skills` |
| Support | `support@securin.io` and `https://github.com/securin-public/securin-skills/issues` |
| Privacy policy | `https://www.securin.io/privacy-policy/` |
| Customer agreement (terms URL on the Claude manifest) | `https://www.securin.io/customer-agreements` |
| OAuth MCP URL | `https://mcp.securin.io/mcp` |
| Logo | `assets/logo.svg` and `assets/logo.png` (official Securin icon on light) |

The root `LICENSE` is Securin proprietary terms. `mcpb/manifest.json` and `mcpb/server/package.json` say `MIT`. That mismatch is recorded here only. Do not change either license in the packaging pull request. Legal must decide what the stores are allowed to display.

No file in the plugin may contain a bearer token, client secret, or customer data.

## Cursor Marketplace and Grok Bot

Submit at `https://cursor.com/marketplace/publish` after this packaging is on a public GitHub repository. The preferred repository is `https://github.com/securin-public/securin-skills`. Grok Bot uses the same Cursor plugin and the same OAuth callbacks. One listing covers the Cursor IDE, Cursor Agents, and Grok Bot.

Manifest: `.cursor-plugin/plugin.json`  
MCP config: root `mcp.json` (`mcpServers.securin.url` = `https://mcp.securin.io/mcp`)  
Skills: discovered from `skills/*/SKILL.md`. `skills/_shared` has no `SKILL.md`.

### Checklist

- [ ] `.cursor-plugin/plugin.json` parses as JSON
- [ ] `name` is `securin-platform` (lowercase kebab-case)
- [ ] `version` is semver `0.1.0`
- [ ] `author.name` is `Securin Inc` and `author.email` is `support@securin.io`
- [ ] `homepage` is `https://www.securin.io`
- [ ] `repository` is `https://github.com/securin-public/securin-skills`
- [ ] `logo` is the relative path `assets/logo.svg`
- [ ] Root `mcp.json` uses the OAuth URL only. No `headers`, no bearer token, no client secret
- [ ] README explains install and first-run OAuth
- [ ] Legal and brand approve listing the Securin name and icon
- [ ] Keycloak accepts `https://www.cursor.com/agents/mcp/oauth/callback` (see blockers)
- [ ] A person has installed the plugin locally and completed SSO once on desktop and once as a Cursor Agent or Grok Bot

Reference: [Cursor plugins](https://cursor.com/docs/plugins) and [plugin manifest reference](https://cursor.com/docs/reference/plugins).

### Human steps

1. Land this packaging on `securin-public/securin-skills` `main` (public).
2. Complete the redirect allowlist in [oauth-redirects.md](oauth-redirects.md).
3. Open `https://cursor.com/marketplace/publish` and submit the repository URL. The plugin manifest is at the repository root, so no subdirectory path is required.
4. Finish Cursor publisher verification.
5. Wait for Cursor review. Do not assume Grok Bot needs a second package.
6. After approval, add the live Marketplace link to the README.

### Blockers before Cursor submission

| Blocker | Status |
| --- | --- |
| Cloud OAuth redirect | Open. `https://www.cursor.com/agents/mcp/oauth/callback` returns `Invalid parameter: redirect_uri`. Desktop `http://localhost:8787/callback` already reaches the login page. Agents and Grok Bot stay unauthenticated until identity adds the https callback. |
| Custom-scheme callback | Open. `cursor://anysphere.cursor-mcp/oauth/callback` is rejected. Some installs still send it. Add it on the Keycloak client if the client allows a custom scheme. |
| Brand and legal | Open. Listing the Securin name and icon in the Cursor Marketplace needs brand and legal sign-off. |
| License text | Open, flag only. Root `LICENSE` is proprietary. `mcpb/manifest.json` and `mcpb/server/package.json` say MIT. The docs landing footer (`docs/index.html`) also links the license as MIT. Confirm the store listing does not claim MIT. The Cursor manifest says `SEE LICENSE IN LICENSE`. The Claude manifest does not set `license`. Do not edit those license strings in this packaging change. |
| Local end-to-end test | Open. Packaging was validated as JSON. A human still needs one live SSO in Cursor desktop and one in a cloud agent or Grok Bot. |

## Claude plugin directory and remote connector

Anthropic now lists two different submissions. Both are required for this product.

1. Plugin bundle, from this Git repository, at `https://claude.ai/directory/manage` (choose Plugin bundle).
2. MCP connector, for the remote server `https://mcp.securin.io/mcp`, at the same portal (choose MCP connector).

People install the unpublished marketplace with:

```text
/plugin marketplace add securin-public/securin-skills
/plugin install securin-platform@securin-skills
```

That flow reads `.claude-plugin/marketplace.json` and the plugin at the repository root. It does not require the public directory. Directory review is a separate step.

Claude Code loads `.mcp.json`. That file is remote HTTP OAuth:

```json
{
  "mcpServers": {
    "securin": {
      "type": "http",
      "url": "https://mcp.securin.io/mcp"
    }
  }
}
```

`mcp-remote` remains the fallback for stdio-only hosts and for the existing Claude Desktop `.mcpb`. The example is `docs/examples/claude-mcp-remote.json`. An unpinned `npx mcp-remote` command in `.mcp.json` would block directory validation, which is why the installed plugin config is the https URL.

Desktop extension listings are deprecated. The directory does not accept a new local `.mcpb` as the connector. Keep the `.mcpb` for the current Claude Desktop download. Expect a reviewer hold because `securin-platform.mcpb` is a binary larger than 256 KiB inside the plugin folder.

### Plugin bundle checklist

- [ ] `.claude-plugin/plugin.json` has `name` `securin-platform`, `version` `0.1.0`, `description`, `author`, and `displayName`
- [ ] `icon` is `./assets/logo.png`
- [ ] `privacyPolicyUrl` is `https://www.securin.io/privacy-policy/`
- [ ] `supportUrl` is an https URL (`https://github.com/securin-public/securin-skills/issues`)
- [ ] `documentationUrl` and `homepage` are https URLs
- [ ] `termsOfServiceUrl` is `https://www.securin.io/customer-agreements` (legal confirms this is the URL to show)
- [ ] `README.md` in the plugin folder is longer than 40 words outside code blocks
- [ ] A `LICENSE` file exists. Do not also set `license` to `MIT` while the root license is proprietary
- [ ] `.mcp.json` is valid JSON, `type` is `http`, and `url` is `https://mcp.securin.io/mcp`
- [ ] No secrets in headers, docs, or examples
- [ ] `.claude-plugin/marketplace.json` lists `securin-platform` with `"source": "./"`
- [ ] `claude plugin validate .` from the repository root, then Validate in the developer portal

References:

- [Submit a plugin](https://claude.com/docs/plugins/submit)
- [Plugin pre-submission checklist](https://claude.com/docs/plugins/pre-submission-checklist)
- [Publish to the directory](https://claude.com/docs/directory/publish)
- [Plugin manifest reference](https://code.claude.com/docs/en/plugins/manifest-reference)
- [Marketplace reference](https://code.claude.com/docs/en/plugins/marketplace-reference)

### MCP connector checklist

- [ ] Server URL `https://mcp.securin.io/mcp` (https, one URL for every user)
- [ ] Authentication: OAuth with Dynamic Client Registration. Metadata does not advertise Client ID Metadata Documents
- [ ] Redirect URIs in [oauth-redirects.md](oauth-redirects.md), including `https://claude.ai/api/mcp/auth_callback` (already accepted) and Claude Code loopback
- [ ] Every tool has a `title` and `readOnlyHint` or `destructiveHint` (server-side, not in this repository)
- [ ] Privacy policy URL, documentation URL, support contact, and icon for the listing form
- [ ] Test account credentials for reviewers, entered only in the portal, never committed
- [ ] Someone has connected the server as a custom connector and called the tools
- [ ] Data handling answers: customer Securin data, per-user OAuth, not a shared bearer token
- [ ] Compliance acknowledgements in the portal

Connector references:

- [Submit a connector](https://claude.com/docs/connectors/building/submission)
- [Authentication for connectors](https://claude.com/docs/connectors/building/authentication)

Anthropic egress range published for connector hosts: `160.79.104.0/21`. Allow that range if a firewall sits in front of `mcp.securin.io` or `auth.securin.io`.

### Human steps

1. Identity completes the redirect list. Re-run the curl checks in [oauth-redirects.md](oauth-redirects.md).
2. Legal confirms the privacy policy URL, the customer-agreement URL, support contact, and permission to list Securin marks.
3. A Securin operator with a paid Claude plan connects GitHub on `https://claude.ai/directory/manage`. The connected account must be able to push to `securin-public/securin-skills`. A fork alone is not enough for the upstream listing.
4. Submit the plugin bundle. Repository `securin-public/securin-skills`, plugin path empty (manifest is at the root), tracked branch `main`.
5. Submit the MCP connector for `https://mcp.securin.io/mcp` with OAuth dynamic client registration.
6. Answer data handling and compliance questions. Provide a reviewer test account in the portal only.
7. Fix any blocking validation finding, then submit for review. Publishing still waits for Anthropic.

Portal contacts if a submission stalls: `directory@anthropic.com` for the plugin, `mcp-review@anthropic.com` for the connector.

### Blockers before Claude submission

| Blocker | Status |
| --- | --- |
| Hosted Claude redirect | Clear on 8 October 2026. `https://claude.ai/api/mcp/auth_callback` already returns the login page. |
| Claude Code loopback | Mostly clear. Any `http://localhost:<port>/callback` and `http://127.0.0.1:<port>/callback` already returns the login page. The portless CIMD forms `http://localhost/callback` and `http://127.0.0.1/callback` are rejected. Add those two so a portless client matches the Anthropic guide. |
| `claude.com` host | Not required. Current Anthropic docs name only `claude.ai`. Those `claude.com` URIs are rejected. Add them only if a capture shows Claude sending them. |
| Public token exchange | Confirm. Authorization server metadata does not list `none` in `token_endpoint_auth_methods_supported`. mcp-remote works today. Confirm `https://mcp.securin.io/token` still accepts the public client after any client edit. |
| Tool annotations | Unknown in this repository. The connector portal flags tools that lack `title` and `readOnlyHint` or `destructiveHint`. That is a server change on `mcp.securin.io`, not a skill change. |
| Reviewer test account | Open. Create a populated Securin account for Anthropic reviewers and paste it only into the portal. |
| GitHub push access | Open. Directory submit requires a Claude organization whose connected GitHub user can push to `securin-public/securin-skills`. |
| License mismatch | Open, flag only. Root `LICENSE` is proprietary. `mcpb/manifest.json` says MIT, and `docs/index.html` labels the license link as MIT. The Claude manifest does not set `license`, so the `LICENSE` file is what the directory reads. Do not claim MIT on the listing. |
| `.mcpb` binary | Expect a reviewer hold. `securin-platform.mcpb` is a binary over 256 KiB in the plugin folder. The directory no longer accepts a new desktop-extension listing. Keep the file for the existing Desktop download path. |
| Brand and legal | Open. Same sign-off as Cursor, plus the privacy policy URL and support contact on the form. |
| Two submissions | Open until both the plugin bundle and the MCP connector are filed. A plugin that points at a remote server still needs the connector listing. |

## What this repository already satisfies

- Cursor manifest and root `mcp.json` are present and use no secrets.
- Claude marketplace file lists `securin-platform` from `./`, so `/plugin marketplace add securin-public/securin-skills` keeps working.
- Claude `.mcp.json` is remote HTTP OAuth, which is the config the directory accepts without an unpinned `npx` launcher.
- Six skills remain the only skill source for both hosts.
- Redirect probe results and the IAM change list are in [oauth-redirects.md](oauth-redirects.md).
