# Where to list the server

Checked against each directory's own docs on 2026-10-31. Rules change often, so re-check before submitting.

**Do these first:** the official MCP Registry, then Glama. Several other directories pull from those two or require them.

**Blocked until the server has OAuth:** the Claude and ChatGPT directories. Neither accepts a pasted API key for a public listing. Both also need a privacy policy and terms page, which advertiserleads.com doesn't have yet.

## Before submitting anywhere

- [ ] `https://advertiserleads.com/mcp` is live and answers `initialize` without a key.
- [ ] Requests with a missing or bad key get **401**, not 403. Once OAuth ships, send `WWW-Authenticate: Bearer resource_metadata=...` with it.
- [ ] Every tool has a `title` and explicit `readOnlyHint`, `destructiveHint` and `openWorldHint` annotations. All four tools here are read-only.
- [ ] advertiserleads.com has a privacy policy, terms, a support contact and a square logo (48px or larger).
- [ ] A test account with data and no MFA is ready for reviewers.

## 1. Official MCP Registry

- [ ] Install `mcp-publisher` from the [registry releases](https://github.com/modelcontextprotocol/registry/releases).
- [ ] Sign in for the `com.advertiserleads` namespace that [server.json](../server.json) uses. Either:
  - **DNS:** add a TXT record on advertiserleads.com: `v=MCPv1; k=ed25519; p=<public key>`. Then run `mcp-publisher login dns --domain advertiserleads.com --private-key <key>`.
  - **HTTP:** serve the key at `https://advertiserleads.com/.well-known/mcp-registry-auth`. Then run `mcp-publisher login http ...`.
- [ ] Run `mcp-publisher publish` from this repo.
- [ ] To switch to a GitHub namespace instead, rename to `io.github.hunterInt/advertiserleads`. Keep the capital I, since the GitHub sign-in is reported to be case-sensitive. Then run `mcp-publisher login github`.
- A remote URL can belong to only one server name, so pick the namespace once.

Sources: [quickstart](https://modelcontextprotocol.io/registry/quickstart), [authentication](https://modelcontextprotocol.io/registry/authentication), [remote servers](https://github.com/modelcontextprotocol/registry/blob/main/docs/modelcontextprotocol-io/remote-servers.mdx)

## 2. Glama (as a connector)

- [ ] Go to [glama.ai/mcp/connectors](https://glama.ai/mcp/connectors), choose **Add MCP Server → Connector**, then enter the name, a short description and the URL. A GitHub repo isn't needed.
- [ ] Add a test API key in the connector's test credentials, so the health check gets past auth. Only connectors that pass the health check are indexed.
- [ ] Claim it in one of two ways:
  - a DNS TXT record at `_glama-claim.advertiserleads.com`
  - a file at `https://advertiserleads.com/.well-known/glama.json`
- Connectors linked to the official registry sync their name, description and URL from it.

Source: [Glama FAQ](https://glama.ai/mcp/faq)

## 3. awesome-remote-mcp-servers

The main `punkpeye/awesome-mcp-servers` list is only for servers you install and run yourself. Hosted servers go in **[punkpeye/awesome-remote-mcp-servers](https://github.com/punkpeye/awesome-remote-mcp-servers)**.

- [ ] Check the server is already listed as a Glama connector. CI checks this.
- [ ] Star the repo, or the PR won't be merged.
- [ ] Open a PR adding this entry, alphabetically, under **Marketing** or **Sales**:

```markdown
- [AdvertiserLeads](https://advertiserleads.com) `https://advertiserleads.com/mcp`
  [![AdvertiserLeads MCP connector](https://glama.ai/mcp/connectors/NAMESPACE/NAME/badges/score.svg)](https://glama.ai/mcp/connectors/NAMESPACE/NAME)
  🔑 - Ecommerce brands that just started, scaled or stopped Meta ads, with store platform and tracking pixels.
```

Replace `NAMESPACE/NAME` with the Glama connector path. Use 🔐 instead of 🔑 once OAuth ships.

Source: [CONTRIBUTING.md](https://raw.githubusercontent.com/punkpeye/awesome-remote-mcp-servers/main/CONTRIBUTING.md)

## 4. Smithery

- [ ] Go to [smithery.ai/new](https://smithery.ai/new) and paste `https://advertiserleads.com/mcp`.
- [ ] If its scan can't get past auth, serve a server card at `https://advertiserleads.com/.well-known/mcp/server-card.json` listing the server info, auth method and tools.
- [ ] Let the `SmitheryBot/1.0` user agent through Cloudflare's bot rules.
- [ ] After it's listed, request vendor verification under **Settings → Verification**.

Source: [Smithery docs](https://smithery.ai/docs/build/external)

## 5. mcp.so

- [ ] Submit at [mcp.so/submit](https://mcp.so/submit) with type **Remote Server**, using this repo's URL.
- Free submissions are reviewed, with no timeline given. A $39 option skips review and adds a verified badge.

Source: [mcp.so/submit](https://mcp.so/submit)

## 6. PulseMCP

- **Submissions are paused** while they rework how listings are imported. Their submit page says to publish to the official registry first.
- [ ] Check [pulsemcp.com/submit](https://www.pulsemcp.com/submit) again once the registry listing is live.

## 7. Claude connectors directory (needs OAuth)

- [ ] Get OAuth 2.0 working with dynamic client registration or CIMD, and PKCE S256.
  - Callback: `https://claude.ai/api/mcp/auth_callback`, plus localhost loopback for Claude Code.
  - Static API-key auth is a limited beta and isn't a path to a public listing.
- [ ] Test it as a custom connector in Claude first. The form asks you to confirm this.
- [ ] Submit at **claude.ai/directory/manage → Submit new → MCP connector** (needs a paid Claude plan). The form asks for:
  - a documentation URL (this repo)
  - privacy policy and support contact
  - an icon
  - a test account
  - seven compliance attestations
- Anthropic's requests come from `160.79.104.0/21`, so let that range through Cloudflare's rules.
- Review is an automatic policy scan, sometimes followed by a human review.

Sources: [submission](https://claude.com/docs/connectors/building/submission), [authentication](https://claude.com/docs/connectors/building/authentication)

## 8. ChatGPT (needs OAuth)

OpenAI's docs now call ChatGPT apps **plugins**.

- [ ] Get OAuth working (DCR, CIMD or a provided client). There's no API-key flow, and public submissions reject server headers.
- [ ] Verify the organization (individual or business), then verify the domain by serving the token at `/.well-known/openai-apps-challenge`.
- [ ] Prepare the listing:
  - a square logo
  - website, support, privacy and terms URLs
  - 5 positive and 3 negative test prompts
  - a video walkthrough
  - a test account without MFA
- [ ] Upload the plugin package at [platform.openai.com/plugins](https://platform.openai.com/plugins). Feedback comes by email, and you publish yourself once it's approved.

Sources: [submission](https://developers.openai.com/apps-sdk/deploy/submission), [guidelines](https://developers.openai.com/apps-sdk/app-submission-guidelines)
