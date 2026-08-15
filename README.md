# SendPage MCP server

Publish an HTML document as a shareable link the recipient can open in one tap —
without leaving the conversation that wrote it.

**Endpoint:** `https://sendpageapp.com/mcp` · **Transport:** Streamable HTTP
(stateless) · **Auth:** bearer API key, free, no signup

---

## Why

An HTML file is a set of drawing instructions, not a document. Sent as an
attachment it arrives as raw source, gets stripped by corporate mail filters as a
known phishing vector, or simply will not open on a phone — which is where most
people read first. Printing to PDF loses interactive charts and quietly cuts the
right-hand columns off wide tables, which is the failure worth fearing: nobody
notices until a client asks about a figure that is not in the document.

A link avoids all of it. The recipient taps once and reads the page as designed,
on whatever device they have.

## Connect it

Generate a free key at **https://sendpageapp.com/docs/mcp**, then:

```bash
claude mcp add --transport http sendpage https://sendpageapp.com/mcp \
  --header "Authorization: Bearer sp_live_..."
```

Or by hand, in `.mcp.json`:

```json
{
  "mcpServers": {
    "sendpage": {
      "type": "http",
      "url": "https://sendpageapp.com/mcp",
      "headers": { "Authorization": "Bearer sp_live_..." }
    }
  }
}
```

## Tools

| Tool | What it does |
|---|---|
| `publish_document` | HTML in, share link out. Returns the link, the document id and an edit token — plus a warning naming any images that would fail to load. |
| `update_document` | Replace the contents or title. **The link does not change**, so a typo found after sending costs a correction rather than a second URL. |
| `list_documents` | Documents published with this key, with links and view counts. |

### The missing-image warning

AI tools emit a page *plus* an `images/` folder, and people paste only the HTML —
so the images 404 and every reader sees broken icons. `publish_document` returns
the offending files **by name**, so they can be inlined as data URIs and the
document updated before the link is handed over. This is the single most common
way a published page is quietly wrong.

## The Skill

[`skills/sendpage/SKILL.md`](skills/sendpage/SKILL.md) teaches Claude when to
offer a link, how to repair a document whose images would not load, and to update
an existing document rather than publish a second one.

```bash
mkdir -p ~/.claude/skills/sendpage
curl -o ~/.claude/skills/sendpage/SKILL.md \
  https://sendpageapp.com/skills/sendpage/SKILL.md
```

## Limits

- **Free: 5 documents a month**, each with a small "Made with SendPage" footer.
  Pro removes both.
- **2 MB per document**, inlined images included.
- **Links are unlisted, not secret.** Anyone with the URL can read the document,
  and `robots.txt` keeps them out of search results. Password protection exists
  for anything confidential, and it gates the exports and the preview thumbnail
  too — not just the page.
- The API key is stored only as a hash. It is shown once when generated and
  cannot be recovered; if you lose it, revoke it and generate another.

## Notes on the transport

Stateless Streamable HTTP: a single `POST` returns one JSON body, and `GET`
answers `405` to decline the optional server-initiated SSE stream. Sessions are
never created, so any instance can serve any request — which is what makes the
server hostable on a serverless runtime, where a long-lived SSE connection would
be killed by the execution limit rather than by anything the client did.

Without a key the endpoint answers **401 with a `WWW-Authenticate` header** —
that is a missing credential, not a broken server.

## Issues

This repository holds the connection metadata and the Skill. For problems with
the server itself, open an issue here.

---

## Use it in DeepSeek Harness (dsh)

This repo is also a DeepSeek Harness **bundle** — install it and SendPage's tools
mount straight into dsh, with no manual MCP config:

```sh
dsh plugin --profile web add https://github.com/jcaiagent7143-ui/sendpage-mcp
```

Then set your free key (get one at https://sendpageapp.com/docs/mcp) so publishing
and exporting work — in the environment, or in `$DSH_HOME/.credentials.yaml`:

```yaml
SENDPAGE_API_KEY: <your key>
```

The tools appear under the `sendpage` namespace — `mcp__sendpage__publish_document`,
`mcp__sendpage__export_document` (png/pdf/docx), and the rest. Prefer to wire the raw
endpoint yourself? It's `https://sendpageapp.com/mcp` (Streamable HTTP).
