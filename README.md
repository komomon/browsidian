<p align="center">
  <img src="img/browsidian.png" alt="Browsidian" width="96" />
</p>

# Browsidian <sup><small>community fork</small></sup>

> **This is a community-maintained fork**, heavily modified from the [original Browsidian](https://github.com/mufeedvh/browsidian) by [mufeedvh](https://github.com/mufeedvh). This version adds session persistence, live file watching, full-text search, code highlighting, font/width toggles, image lightbox, and many bug fixes — with the goal of making Browsidian practical for daily Obsidian use behind a reverse proxy.

A **zero-dependency** web app to browse, search, and edit an [Obsidian](https://obsidian.md) vault directly in your browser. No Electron, no npm build step — just a single Node.js server and vanilla JavaScript.

> Host it behind Caddy/Nginx, pick a local folder with the File System Access API, connect Dropbox, or try the in-browser demo vault. All four modes share the same UI.

![Screenshot](img/screenshot.png)

## What's different from upstream

This fork adds substantial features and fixes beyond the original:

| Area | Upstream | This fork |
|------|----------|-----------|
| **Session** | Lost on refresh | Tabs, expanded dirs, active file all restored |
| **File watching** | Manual refresh only | Auto-detects external changes every 5s |
| **Content search** | File name only | Full-text search across all `.md` files |
| **Code blocks** | Plain text | Syntax highlighting + copy button |
| **Display** | Fixed | Font size toggle (S/M/L) + content width (full/medium/narrow) |
| **Images** | Inline only | Click-to-zoom lightbox |
| **Tree** | No refresh | Manual refresh button + auto-polling |
| **Bug fixes** | — | 15+ bugs fixed (XSS, polling, state races, edge cases) |

## Why this fork

- **Zero dependencies** — the frontend is vanilla JS (no React, no bundler), the backend is raw Node.js `http` (no Express). Only highlight.js is loaded from CDN for code blocks.
- **Four vault backends** — local disk (server mode), folder picker (browser mode), Dropbox (OAuth), or in-browser demo (localStorage). Switch anytime.
- **Session persistence** — open tabs, expanded directories, and the active file survive page refreshes (localStorage).
- **Live file watching** — detects files created, modified, or deleted by another Obsidian client within 5 seconds (polling with SHA1 directory hashes).
- **Obsidian-native** — wikilinks `[[Note]]`, `[[Note|Alias]]`, relative image paths `![](attachments/img.png)`, and `#tags` all work.
- **Responsive layout** — sidebar tree, tabbed editor, outline panel, and status bar scale down to mobile widths.

## Four vault modes

| Mode | Storage | Best for |
|------|---------|----------|
| **Server** | Local disk via Node.js API | Self-hosting behind a reverse proxy |
| **Browser** | Local folder via File System Access API | Quick access, no server vault configured |
| **Dropbox** | Cloud via OAuth + Dropbox API | Remote vault, cross-device access |
| **Demo** | In-browser localStorage | Testing, automation agents, browsers without folder picker |

## Features

### Vault browsing
- Tree view with expand/collapse directories
- **File name search** with instant client-side filtering across the full vault tree
- **Full-text content search** across all `.md` files with match previews (server + local modes)
- Manual **Refresh** button to re-sync the tree from disk
- Click a **folder name** to select it as the default destination for new files/folders
- Hidden directories: `.obsidian`, `.git`, `node_modules`, `.trash`, `.DS_Store`
- **Drag & drop** files onto folders to move them (confirmation dialog)
- Right-click **context menu** to delete files

### Editing
- Multi-tab editor — open several notes at once, switch with a click, close with `x` or **Ctrl+W**
- **Auto-save** after ~1.2s of inactivity + manual **Ctrl+S** / **Save** button
- Markdown **preview mode** with live HTML rendering
- Click the preview to switch to editing; click away to return to preview
- New files are automatically given `.md` extension
- New files/folders default to the currently selected directory

### Session persistence
- Open tabs, expanded directories, active file, and selected folder are saved to `localStorage`
- **Full state restoration** on page refresh — pick up exactly where you left off
- Editor/preview view mode remembered per tab

### Live file watching
- Polls expanded directories every 5 seconds (no WebSocket needed)
- Server mode: batched SHA1 hash comparison (single HTTP request)
- Browser/Dropbox mode: directory re-listing with entry-level diff
- Detects external file **creation, deletion, and modification**
- Automatically closes tabs for externally deleted files

### Markdown preview
- Headings (`#` to `######`) with auto-generated anchor IDs
- Bold, italic, inline code, fenced code blocks
- Blockquotes, horizontal rules, ordered/unordered lists
- Tables (GitHub-flavored, header + separator row)
- Obsidian **wikilinks**: `[[Note]]`, `[[Note|Alias]]` (click to navigate, cross-directory resolution)
- Obsidian **tags**: `#tag`, `#tag/sub-tag`
- Relative **images**: `![](attachments/img.png)` resolves from the note's directory
- External images and links open in new tabs

### Code blocks
- **Syntax highlighting** via highlight.js (auto-detected language)
- **Copy button** on hover — one click copies the code block to clipboard

### Outline panel
- Right-side panel showing document headings (collapsible)
- Click any heading to jump to it (smooth scroll in preview, cursor jump in editor)
- Panel can be collapsed via the **Outline** toggle button

### Display customization
- **Font size**: small / medium / large (persisted)
- **Content width**: full / medium (960px) / narrow (720px) for focused reading (persisted)
- **Dark / Light** theme toggle (persisted)
- All preferences survive page refreshes

### Image lightbox
- Click any image in the preview to open it full-size
- **Esc** or click the overlay to close

### UI details
- Flat, consistent SVG icon set (zero external icon dependencies)
- Status bar with app version, author credit, and GitHub link
- Subtle animations: hover states, toggle transitions, button feedback
- Proper sub-path reverse proxy support (`/obobob/` behind Caddy/Nginx)

## Requirements

- Node.js 18+
- For **Browser mode**: Chrome / Edge / Brave (File System Access API)

## Getting started

### Server mode (recommended for full Obsidian vault access)

```bash
node server.js --vault /path/to/your/vault
# or
OBSIDIAN_VAULT=/path/to/your/vault node server.js
```

Open: `http://127.0.0.1:5173`

Custom port and host:

```bash
node server.js --vault /path/to/vault --port 8080 --host 0.0.0.0
```

### Reverse proxy (Caddy example)

```
handle_path /obsidian/* {
    reverse_proxy http://127.0.0.1:5173
}
```

The app auto-detects its base path from the `<base>` tag — no extra configuration needed.

### Browser mode (no server vault)

Start the server without a vault:

```bash
node server.js
```

Open the app, click **Choose local vault**, and select your Obsidian folder. The browser requests read/write permission once.

### Demo mode

Click **Try demo vault** in the startup dialog. Everything stays in `localStorage` — useful for testing or automation agents.

### Dropbox mode

Set environment variables on the server:

- `DROPBOX_APP_KEY`
- `DROPBOX_APP_SECRET`
- `DROPBOX_REDIRECT_URI` (must match your Dropbox app's redirect URI whitelist)

Then click **Connect Dropbox** in the app, complete the OAuth flow, and pick a vault folder using the built-in Dropbox folder navigator.

Dropbox file operations are proxied through `/api/dropbox/files/*` to avoid browser CORS limitations.

## Production

A hosted instance is available at **[browsidian.app.lamouche.fr](https://browsidian.app.lamouche.fr/)** (Browser + Demo modes only; Server and Dropbox modes require running your own server).

## Security model

- **Path traversal protection**: all file operations are constrained to the vault root (resolved realpath + prefix check).
- **Hidden directories**: `.obsidian`, `.git`, `node_modules`, `.trash`, `.DS_Store` are excluded.
- **Demo mode**: all data stays in the browser's `localStorage` — no disk or network access.
- **XSS prevention**: all user-generated Markdown content is HTML-escaped before rendering.
- This app is designed to run on your local machine or a private server. Do not expose it to the public internet without authentication.

## Markdown preview coverage

| Element | Support |
|---------|---------|
| Headings (`#` – `######`) | Full, with auto-generated IDs for outline navigation |
| Bold / Italic | `**bold**`, `*italic*` |
| Inline code | `` `code` `` |
| Fenced code blocks | ` ```language ``` ` with syntax highlighting |
| Blockquotes | `>` single-level |
| Horizontal rules | `---`, `***` |
| Unordered lists | `-`, `*` |
| Ordered lists | `1.`, `2.` |
| Tables | GitHub-style with header separator |
| Links | `[text](url)` external + `[[wikilinks]]` internal |
| Images | `![alt](path)` with relative vault path resolution |
| Tags | `#tag`, `#tag/sub-tag` |
| Task lists | Not yet supported |
| Footnotes | Not yet supported |
| Callouts | Not yet supported |

## Troubleshooting

- **Old UI after update**: hard-refresh the page (Ctrl+Shift+R) and restart the server.
- **Choose local vault grayed out**: use Chrome / Edge / Brave served from `http://127.0.0.1`.
- **Can't edit in Browser mode**: accept the read/write permission prompt from the browser.
- **Dropbox connect fails**: check the status bar for the backend error message. Verify the redirect URI is whitelisted in your Dropbox app.

## Contributing

1. Branch from `main`: `git checkout -b features/my-feature`
2. Keep changes focused, use clear commit messages
3. Push and open a PR from your branch → `main`
4. Describe the change, add screenshots for UI changes, and list manual checks

## Credits

- **Original Browsidian** by [mufeedvh](https://github.com/mufeedvh) — the foundation this fork is built on
- **This fork** maintained by [komomon](https://github.com/komomon)
- Built with <span style="color:#36d399;">♥</span> and vanilla JavaScript

## License

MIT — same as upstream.
