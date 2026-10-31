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

This repository holds the setup guide, config snippets and registry metadata. The server itself is hosted by AdvertiserLeads, so there's nothing to install or run.

> **Status: early.** The endpoint, tool names and limits below are v1 and may still change before launch. OAuth sign-in is coming; until then, connect without a key or with an API key header.

## Connect

| | |
| --- | --- |
| Endpoint | `https://advertiserleads.com/mcp` |
| Transport | Streamable HTTP |
| Auth | Optional. Send `Authorization: Bearer al_...` with an API key from [Settings](https://advertiserleads.com/app/settings?utm_source=github&utm_medium=readme&utm_campaign=mcp). Without a key it works anonymously, with lower limits. |

### Claude Code

Without a key:

```bash
claude mcp add --transport http advertiserleads https://advertiserleads.com/mcp
```

With your API key:

```bash
claude mcp add --transport http advertiserleads https://advertiserleads.com/mcp --header "Authorization: Bearer al_YOUR_KEY"
```

### Claude (claude.ai and the desktop app)

1. Open **Settings → Connectors** and choose **Add custom connector**.
2. Name it `AdvertiserLeads` and paste `https://advertiserleads.com/mcp` as the URL.
3. In a chat, turn it on from the tools menu.

Custom connectors sign in with OAuth, not a pasted key, so until OAuth ships this connects anonymously. To use your plan's limits in Claude Desktop today, use the config in [examples/claude_desktop_config.json](examples/claude_desktop_config.json), which passes your key through [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) (needs Node.js).

### ChatGPT

1. Open **Settings → Apps & Connectors → Advanced settings** and turn on **Developer mode** (Plus, Pro, Business, Enterprise or Edu).
2. Back in **Apps & Connectors**, choose **Create**, name it `AdvertiserLeads`, paste `https://advertiserleads.com/mcp`, and pick **No authentication**.
3. In a chat, select it from the **+** menu under Developer mode.

Like Claude, ChatGPT connects anonymously until OAuth ships.

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

Any client that supports remote MCP servers over Streamable HTTP can use `https://advertiserleads.com/mcp`, with the optional `Authorization` header.

## Tools

| Tool | What it does | Inputs |
| --- | --- | --- |
| `search_signals` | Today's lead signals: brands that are newly advertising, scaling, struggling or stopped | `niche`, `signal`, `min_ads`, `platform` (store platform), `pixel` (tracking pixel or tool), `q` (text search), `limit`, `offset` |
| `lookup_brand` | One brand by domain: when it started advertising, live ads and trend | `domain` |
| `check_store` | A site's store platform and the tracking pixels and marketing tools it loads | `domain` |
| `list_niches` | The niches we track, with their ids for `search_signals` | none |

Niches tracked today: automotive aftermarket (`auto_aftermarket`), supplements (`supplements`) and skincare (`skincare`).

### What you get on each plan

The server follows the same plan rules as the website, so it never shows more than your plan does there.

| | No key | Free account | Paid plans |
| --- | --- | --- | --- |
| `search_signals` | Masked brands: signal, niche, ad counts, channels and store are visible; name, domain and Ad Library link are hidden | Masked, as on the free feed | Full rows, counted toward your plan's monthly export rows |
| `lookup_brand` | Advertising since, live ads and trend | Full result for 5 new domains a day | Full result; Pro and up also get ad history |
| `check_store`, `list_niches` | Yes | Yes | Yes |

Lookups of brands we haven't seen before cost a live Ad Library check, so they have a daily limit. [Plans and pricing](https://advertiserleads.com/?utm_source=github&utm_medium=readme&utm_campaign=mcp#pricing): Free, Starter $49, Pro $99, Agency $249 a month, and custom Data / API plans.

## Example prompts

**Agencies**
- "Find automotive aftermarket brands that started advertising on Meta this week. Keep the Shopify stores with Klaviyo installed, and draft a three-line intro email for each offering to take over their Meta ads."
- "Which skincare brands are scaling right now with more than 10 live ads? Group them by store platform."

**UGC creators**
- "List supplement brands that are newly advertising and still have under 5 live ads. Those are the ones that need more creative. Write me a short DM pitching a UGC test package to each."
- "Look up gymshark.com and tell me how long they've been advertising and whether their ad count is going up."

**SaaS and app sellers**
- "Find Shopify stores in auto aftermarket that are scaling but don't have Klaviyo. Draft an email pitching our email-marketing app."
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
