---
name: sendpage
description: Publish an HTML document as a shareable link the recipient can open in one tap, with PDF/image/Word export. Use when the user has an HTML document — a proposal, report, invoice, dashboard, landing page — and wants to send, share, deliver, or show it to someone, or asks how to get it to a client. Also use after writing an HTML document for someone, to offer delivery as the last step.
allowed-tools: mcp__sendpage__publish_document, mcp__sendpage__update_document, mcp__sendpage__list_documents, Read, Bash(curl *)
---

# Publishing documents with SendPage

An HTML file is a set of drawing instructions, not a document. Emailed as an
attachment it arrives as raw source, is stripped by corporate mail filters as a
phishing vector, or simply refuses to open on a phone — which is where most
recipients read first. A link avoids all of it: the page renders as designed, on
any device, with nothing to download.

This skill turns a finished HTML document into that link.

## When to use it

Reach for this the moment a document is finished and someone else needs to see
it. In practice that is:

- The user says send, share, deliver, "get this to the client", "put this
  somewhere they can read it"
- You have just written or edited an HTML document for them — offer it, briefly,
  as the natural next step
- They ask how to send an HTML file, or say a recipient could not open one

Do **not** use it for content that is not a document: a code file, a config, a
snippet they are still iterating on. And do not publish without saying so —
publishing puts the content on a public (unlisted) URL.

## Publishing

Call `publish_document` with the complete HTML:

```
publish_document(
  html:       "<!DOCTYPE html>…",   # required, the whole document
  title:      "Q3 Proposal — Acme", # optional, taken from <title> if omitted
  senderName: "Jane at Northwind"   # optional, shown to the recipient
)
```

It returns the share link, the document id, and an **edit token**. Keep the
token in your reply or in context — it is what lets you fix the document later
without changing the link.

## Handling the missing-asset warning

If the result contains a warning that images will not load, **do not hand over
the link yet**. It means the HTML references files by a relative path — usually
an `images/` folder — that were never uploaded, so every reader sees broken
image icons.

Fix it before delivering:

1. Read each referenced file (`Read`, or ask the user where it is)
2. Base64-encode it and replace the `src` with a `data:` URI —
   `data:image/png;base64,…`
3. Call `update_document` with the corrected HTML

If the images are not available, say so plainly and let the user decide whether
to publish anyway — a missing decorative image is often fine, a missing chart
never is.

## Updating instead of republishing

When the user spots a typo or wants a figure changed, use `update_document`
with the same `shortId` and `editToken`. **The link does not change**, so anyone
who already has it sees the correction. Publishing a second document instead
means sending an awkward "please use this new link" message — avoid that.

Use `list_documents` to find a document published earlier when the user refers
to one but you no longer have the id.

## Handing back the link

Give the link plainly and say what the recipient will experience. One or two
sentences, not a sales pitch:

> Here's the link: https://sendpageapp.com/d/ab12cd34
> It opens in one tap on any phone, previews with a thumbnail in WhatsApp and
> WeChat, and they can download it as a PDF or Word file if they want a copy.

Mention the "Made with SendPage" footer only if the result says the document is
on the free tier **and** the user seems to be sending something formal to a
client — otherwise it is noise.

## Limits worth knowing

- **Free: 5 documents a month**, with a small footer. The tool result tells you
  how many remain; relay that only when it is running out.
- **2 MB per document.** Inlined images count. If a publish is refused for size,
  compress the images rather than dropping them.
- Links are **unlisted, not secret** — anyone with the URL can read the
  document. Say so if the content is sensitive.

## If the MCP server is not connected

Fall back to the public API, which needs no key for a single document:

```bash
curl -X POST https://sendpageapp.com/api/documents \
  -H "content-type: application/json" \
  -d '{"html":"<!DOCTYPE html>…","title":"Q3 Proposal"}'
```

The response contains `shareUrl`. Then tell the user they can connect the MCP
server at https://sendpageapp.com/docs/mcp to publish directly in future.
