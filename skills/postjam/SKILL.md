---
name: postjam
description: Post reels and slideshows to TikTok, Instagram, YouTube, Facebook and Threads through Postjam, once or on a schedule. Use when the person wants to remix a TikTok or Instagram post, turn a winning post into a format that keeps posting, post slides or a video made here, download TikTok/Pinterest/YouTube links into a gallery, schedule or bulk-schedule posts, connect a social account, or see how their posts did.
---

# Postjam

Postjam is a hosted service the person signs into. Everything here goes through its MCP server at `https://mcp.postjam.app`. The server explains its own tools and flow when it connects, so this file only covers getting connected and the few rules that matter before anything goes out.

## 1. Connect the MCP server

Check whether this agent already has tools from a server named `postjam` (`workspace_status` is one of them). If it does, skip to step 2.

If not, add it. It is a remote HTTP server with no API key. The first call opens a browser sign-in to postjam.app, and a new person can sign up there.

- **Claude Code:** `claude mcp add --transport http postjam https://mcp.postjam.app`
- **Codex:** `codex mcp add postjam --url https://mcp.postjam.app`
- **Cursor:** add `"postjam": { "url": "https://mcp.postjam.app" }` under `mcpServers` in `.cursor/mcp.json` (or `~/.cursor/mcp.json` for every project).
- **Claude Desktop or claude.ai:** the person adds it themselves under Customize → Connectors → Add custom connector, with that URL.
- **Any other agent:** use its own way of adding a remote HTTP MCP server, named `postjam`, at that URL.

Then reconnect or restart as the agent requires, finish the sign-in if asked, and confirm the tools are listed. If the agent has to be restarted before the tools show up, say so and stop. Do not try to call Postjam's HTTP endpoint yourself.

## 2. Start from the workspace

Call `workspace_status` first. It says what the workspace holds, the one next step, and the questions to ask the person. Ask them one at a time.

Before making anything, find out whether the person wants **one post** or **a series**:

- **One post** of something they have, or that you make here, is an export with no format: upload the files (`prepare_upload` → run the commands it gives → `finish_upload`), then `create_export`, then `post_exports`.
- **A series** is a format with a schedule: `import_post` from a TikTok or Instagram link (or `create_format` from scratch), then `bind_accounts`, then `set_schedule`.

The server's prompts (`start`, `remix`, `download`, `post`) are each of these flows cut to one run. Follow one when it fits.

## 3. Rules

- **Nothing goes out without a yes.** `bind_accounts`, `set_schedule` and `post_exports` publish to real accounts. Read back the accounts, days, times and timezone, and wait for an explicit yes before calling them.
- **The browser is the person's.** Platform sign-ins (`connect_account` hands back a link or a QR code), choosing a plan, and payment all happen in their browser. Pass the link on. Never ask for a password or card.
- **Ask where they will sign in.** Before `connect_account`, unless the person has already said, ask whether they will sign in on this device (`how: link`, a link to open here) or on their phone (a QR code to scan). TikTok usually wants the phone, since its app holds the account.
- **A phone gets a QR image.** The text QR `how: qr` returns only scans in a terminal's monospace font. When the person signs in on a phone, render the link as a QR image (any local QR encoder, as a PNG with a white margin) and show it inline in the chat, with the link under it as a fallback. Every link works once and expires in 15 minutes, so start a new `connect_account` if they say it ran out.
- **When a limit is hit**, for example remixes used up or the trial's posts spent, say so and pass on the link the tool returns.
- **Video remixes** need footage. When `import_post` says the link is a video, pass on its advice and ask whether the person will upload their own clips or generate them, before making anything.
