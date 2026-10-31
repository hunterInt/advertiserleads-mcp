# AdvertiserLeads MCP server

Ask Claude, ChatGPT, Cursor or any MCP client which ecommerce brands just started running Meta ads, which ones are scaling, and what each brand's store runs on, then go straight to research and outreach in the same chat.

[AdvertiserLeads](https://advertiserleads.com/?utm_source=github&utm_medium=readme&utm_campaign=mcp) checks ecommerce brands' US Meta ads every day and turns the changes into lead signals:

| Signal | Meaning |
| --- | --- |
| `newly_advertising` | The brand's first ad is recent and we found no older ones in the Ad Library |
| `scaling` | Its number of live ads is growing |
| `struggling` | Its ads are being cut back or dropping off |
| `stopped` | It stopped advertising |

Each brand also comes with its store platform (Shopify and others) and the tracking pixels and marketing tools on its site: Meta, Google Ads, GA4, GTM, TikTok, Pinterest, Snapchat, Reddit, Bing, Klaviyo, Triple Whale, Northbeam, Elevar, Attentive and Postscript.

**Any store, any category.** Brand lookups and store checks work on any ecommerce domain you give them, whatever it sells. The daily signal feed covers a growing list of categories (automotive aftermarket, supplements, skincare, pet, home & kitchen, with more added every month). See [Coverage](#coverage).

This repository holds the setup guide, config snippets and registry metadata. The server itself is hosted by AdvertiserLeads, so there's nothing to install or run.

> **Status: early.** The endpoint, tool names and limits below are v1 and may still change before launch.

## Connect

| | |
| --- | --- |
| Endpoint | `https://advertiserleads.com/mcp` |
| Transport | Streamable HTTP |
| Auth | Optional. Sign in with your AdvertiserLeads account over OAuth 2.1 (Claude, ChatGPT, Claude Code and other OAuth-capable clients), or send `Authorization: Bearer al_...` with an API key from [Settings](https://advertiserleads.com/app/settings?utm_source=github&utm_medium=readme&utm_campaign=mcp). Without either it works anonymously, with lower limits. |

Signing in and an API key give the same result: your account's plan and limits. When an anonymous client hits a limit that an account raises, the server answers `401` with a `WWW-Authenticate` challenge, so clients that support OAuth offer to sign in and then retry. You can see and disconnect signed-in apps under **Connected apps** in [Settings](https://advertiserleads.com/app/settings?utm_source=github&utm_medium=readme&utm_campaign=mcp).

### Claude Code

```bash
claude mcp add --transport http advertiserleads https://advertiserleads.com/mcp
```

It starts anonymously. To sign in, run `/mcp` in Claude Code, pick `advertiserleads` and choose **Authenticate**; your browser opens the AdvertiserLeads sign-in page.

Or use an API key instead of signing in:

```bash
claude mcp add --transport http advertiserleads https://advertiserleads.com/mcp --header "Authorization: Bearer al_YOUR_KEY"
```

### Claude (claude.ai and the desktop app)

1. Open **Settings → Connectors** and choose **Add custom connector**.
2. Name it `AdvertiserLeads` and paste `https://advertiserleads.com/mcp` as the URL. Leave the OAuth client fields empty.
3. In a chat, turn it on from the tools menu.

Claude uses the tools anonymously at first. When you hit a limit an account raises, Claude shows a **Connect** button; sign in with your AdvertiserLeads email and Claude retries the call with your plan.

### ChatGPT

1. Open **Settings → Apps & Connectors → Advanced settings** and turn on **Developer mode** (Plus, Pro, Business, Enterprise or Edu).
2. Back in **Apps & Connectors**, choose **Create**, name it `AdvertiserLeads`, paste `https://advertiserleads.com/mcp`, and pick **OAuth**. ChatGPT registers itself; no client ID or secret is needed.
3. Sign in with your AdvertiserLeads email when asked, then select it from the **+** menu under Developer mode in a chat.

### Cursor

Add this to `~/.cursor/mcp.json` (or `.cursor/mcp.json` in a project). See [examples/cursor-mcp.json](examples/cursor-mcp.json).

```json
{
  "mcpServers": {
    "advertiserleads": {
      "url": "https://advertiserleads.com/mcp",
      "headers": { "Authorization": "Bearer al_YOUR_KEY" }
    }
  }
}
```

### VS Code (GitHub Copilot agent mode)

Add this to `.vscode/mcp.json`. VS Code asks for the key once and stores it securely. See [examples/vscode-mcp.json](examples/vscode-mcp.json).

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "advertiserleads-key",
      "description": "AdvertiserLeads API key (starts with al_)",
      "password": true
    }
  ],
  "servers": {
    "advertiserleads": {
      "type": "http",
      "url": "https://advertiserleads.com/mcp",
      "headers": { "Authorization": "Bearer ${input:advertiserleads-key}" }
    }
  }
}
```

### Anything else

Any client that supports remote MCP servers over Streamable HTTP can use `https://advertiserleads.com/mcp`, signed in or with the optional `Authorization` header.

For OAuth clients, the server follows the [MCP authorization spec](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization):

| | |
| --- | --- |
| Protected resource metadata | `https://advertiserleads.com/.well-known/oauth-protected-resource/mcp` |
| Authorization server metadata | `https://advertiserleads.com/.well-known/oauth-authorization-server` |
| Client registration | Dynamic Client Registration (RFC 7591) or a Client ID Metadata Document URL |
| Flow | Authorization code with PKCE (S256 only), rotating refresh tokens, revocation (RFC 7009) |
| Redirect URIs | Any https URI, an app's own URI scheme, or a loopback `http://localhost` / `http://127.0.0.1` address on any port |
| Scope | `mcp` |

## Tools

| Tool | What it does | Inputs |
| --- | --- | --- |
| `search_signals` | Today's lead signals: brands that are newly advertising, scaling, struggling or stopped | `niche`, `signal`, `min_ads`, `platform` (store platform), `pixel` (tracking pixel or tool), `q` (text search), `limit`, `offset` |
| `lookup_brand` | One brand by domain: when it started advertising, live ads and trend | `domain` |
| `check_store` | A site's store platform and the tracking pixels and marketing tools it loads | `domain` |
| `list_niches` | The niches we track, with their ids for `search_signals` | none |

Without signing in or on the free plan, `search_signals` only filters by `niche`; `signal`, `min_ads`, `platform`, `pixel` and `q` need a paid plan.

### Coverage

- **`lookup_brand` and `check_store` work for any store**, in any category. Give them a domain and they check it, even if we've never seen it before.
- **`search_signals` covers the categories we crawl every day:**

  | Niche | id |
  | --- | --- |
  | Automotive aftermarket | `auto_aftermarket` |
  | Supplements | `supplements` |
  | Skincare | `skincare` |
  | Pet | `pet` |
  | Home & kitchen | `home_kitchen` |

  A new niche starts returning results once it tracks 50 confirmed stores, and we add new niches every month. `list_niches` always returns the current list.
- **Don't see yours?** [Request a niche](https://advertiserleads.com/?utm_source=github&utm_medium=readme&utm_campaign=mcp#feed) and we'll email you when it's live.

### What you get on each plan

The server follows the same plan rules as the website, so it never shows more than your plan does there.

| | Not signed in, no key | Free account | Paid plans |
| --- | --- | --- | --- |
| `search_signals` | Masked brands: signal, niche, ad counts, channels and store are visible; name, domain and Ad Library link are hidden. Only the `niche` filter applies | Masked, as on the free feed. Only the `niche` filter applies | Full rows, counted toward your plan's monthly export rows. Every filter applies |
| `lookup_brand` | Advertising since, live ads and trend | Full result for 5 new domains a day | Full result; Pro and up also get ad history |
| `check_store`, `list_niches` | Yes | Yes | Yes |

Lookups of brands we haven't seen before cost a live Ad Library check, so they have a daily limit. [Plans and pricing](https://advertiserleads.com/?utm_source=github&utm_medium=readme&utm_campaign=mcp#pricing): Free, Starter $49, Pro $99, Agency $249 a month, and custom Data / API plans.

## Example prompts

Swap in your own category or list of domains; the brand and store tools aren't limited to the niches below.

**Anyone**
- "Here are 20 stores I'm prospecting: [domains]. Which ones run Meta ads right now, since when, and is their ad count going up or down?"
- "Is this brand still advertising? Check brand.com and tell me what store platform and marketing tools it uses."

**Agencies**
- "Find automotive aftermarket brands that started advertising on Meta this week. Keep the Shopify stores with Klaviyo installed, and draft a three-line intro email for each offering to take over their Meta ads."
- "Which skincare brands are scaling right now with more than 10 live ads? Group them by store platform."
- "Find pet brands that just started running Meta ads and use the TikTok pixel too. Those already test new channels, so draft a pitch for our TikTok Shop service."

**UGC creators**
- "List supplement brands that are newly advertising and still have under 5 live ads. Those are the ones that need more creative. Write me a short DM pitching a UGC test package to each."
- "Look up gymshark.com and tell me how long they've been advertising and whether their ad count is going up."

**SaaS and app sellers**
- "Find Shopify stores in home & kitchen that are scaling but don't have Klaviyo. Draft an email pitching our email-marketing app."
- "Check these five domains and tell me which ones run the TikTok pixel and which use Triple Whale or Northbeam."

**Lenders and revenue-based financing**
- "Which brands across all niches are scaling fastest this week? Those may need working capital for inventory. Give me a shortlist with store platform and live ad counts."
- "Show me skincare brands that stopped advertising in the last month, and check which of them are still on Shopify."

## About the data

- Ads come from the Meta Ad Library, US ads only, checked daily. Busier brands are checked daily; quieter ones every 3 or 7 days.
- "Newly advertising" is checked against the brand's full ad history, so a brand that paused and came back isn't counted as new.
- Store platform and pixels come from loading the brand's own site.

More at [advertiserleads.com](https://advertiserleads.com/?utm_source=github&utm_medium=readme&utm_campaign=mcp).

## Links

- [Website](https://advertiserleads.com/?utm_source=github&utm_medium=readme&utm_campaign=mcp)
- [Brand lookup](https://advertiserleads.com/lookup?utm_source=github&utm_medium=readme&utm_campaign=mcp)
- [Free Facebook Ad Library scraper](https://advertiserleads.com/tools/facebook-ad-library-scraper?utm_source=github&utm_medium=readme&utm_campaign=mcp)
- [Where this server is listed](docs/directories.md)

## License

The docs and config snippets in this repository are MIT licensed. See [LICENSE](LICENSE). The license doesn't cover the AdvertiserLeads service or its data.
