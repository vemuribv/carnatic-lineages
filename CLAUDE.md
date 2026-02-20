# CLAUDE.md — Carnatic Lineages

This file provides guidance for AI assistants working on this repository.

## Project Overview

**carnatic-lineages** is a single-file, client-side interactive visualization of the Carnatic music Guru-Shishya Parampara (teacher-student tradition), specifically the Tyagaraja lineage. It renders a D3.js force-directed network graph showing ~120 musicians across ~9 generations spanning roughly 1767 to the present.

The entire application lives in **one file**: `index.html`. There is no build step, no framework, no package manager, and no backend.

## Repository Structure

```
carnatic-lineages/
├── index.html      # The entire application (HTML + CSS + JS + data)
└── README.md       # Minimal — just the project title
```

### index.html Internal Layout

| Lines     | Section                         | Description                                                    |
|-----------|---------------------------------|----------------------------------------------------------------|
| 1–8       | HTML boilerplate / CDN imports  | D3.js v7.8.5 and Google Fonts (Playfair Display, Source Sans 3) |
| 9–107     | `<style>` — CSS                 | Dark-themed UI (GitHub-inspired), layout, panels, animations  |
| 108–182   | `<body>` HTML markup            | Header, SVG graph container, detail panel, issues panel, modal |
| 183–626   | `<script>` — JavaScript         | All application logic (data, D3 graph, interactions, issues)  |

### JavaScript Sections (within `<script>`)

| Lines (approx.) | Purpose                                                                 |
|-----------------|-------------------------------------------------------------------------|
| 184–620         | `DATA` array — 120 musician objects (the dataset)                       |
| 213–273         | D3 force simulation setup, node/link rendering                          |
| 252–256         | Click and hover handlers for node interaction                           |
| 254–256         | Hover highlighting — subtree dimming/highlighting                       |
| 326–391         | `showDetail(d)` — populates the right-side detail panel                |
| 393–396         | `focusNode(d)` — zooms the viewport to a selected node                 |
| 399–417         | Search — real-time autocomplete against musician names                  |
| 420–440         | Filter buttons — by language tradition or award winners                 |
| 449–452         | Drag handlers for repositioning nodes                                   |
| 455             | Initial zoom/translate for graph starting position                      |
| 457–469         | Issue tracking system setup and `APPS_SCRIPT_URL` constant             |
| 471–626         | Issue reporting logic (localStorage, Google Apps Script, UI handlers)   |

## Data Schema

Each entry in the `DATA` array is an object with these fields:

```js
{
  id: "unique_string",       // e.g., "tyagaraja", "manambuchavadi"
  name: "Full Name",
  birth: 1767,               // year (number), or null
  death: 1847,               // year (number), or null/omit if living
  place: "City, State",      // birth/primary location
  language: "Telugu",        // tradition language: "Telugu" | "Tamil" | "Kannada" | "Malayalam" | "Trinity"
  generation: 0,             // 0 = Tyagaraja himself; higher = further down the lineage
  guru_ids: "id1,id2",       // comma-separated IDs of this musician's gurus (builds graph edges)
  sk: 1947,                  // Sangeetha Kalanidhi award year, or null
  sna: 1954,                 // Sangeet Natak Akademi award year, or null
  other_awards: "...",       // free text for other awards, or null
  bio: "...",                // short biographical description
  url: "https://..."         // Wikipedia or other reference URL, or null
}
```

**Node colors by `language` field:**
- `"Telugu"` → red (`#e05c5c`)
- `"Tamil"` → blue (`#58a6ff`)
- `"Kannada"` → green (`#3fb950`)
- `"Malayalam"` → purple (`#a371f7`)
- `"Trinity"` → gold (`#d4a843`) — reserved for the three Carnatic Trinity composers

**Node sizing:** Base radius scales with `generation` (generation 0 is largest). Nodes with `sk` or `sna` awards get a larger stroke ring.

## Key Configuration

**`APPS_SCRIPT_URL`** (line 469 in `index.html`): The Google Apps Script deployment URL for routing reported issues to a Google Sheet. This is the only configurable constant in the project. It is hardcoded in the file.

To set up your own issue-routing sheet:
1. Create a Google Sheet → Extensions → Apps Script → paste the `doPost` function shown in the comment block at lines 458–468.
2. Deploy as a Web App with "Execute as: Me" and "Access: Anyone."
3. Replace the URL at line 469 with your deployment URL.

## Running the Application

No build step is needed.

```bash
# Option 1: Open directly in browser
open index.html

# Option 2: Serve via Python (avoids some browser CORS quirks)
python -m http.server 8000
# Then visit http://localhost:8000

# Option 3: Any static file server works
npx serve .
```

## Development Conventions

### Making Data Changes

All musician data lives in the `DATA` array in `index.html`. To add or update a musician:
1. Add/edit the object in the `DATA` array.
2. Set `guru_ids` to a comma-separated list of existing `id` values for the musician's teachers — this is what creates edges in the graph.
3. Choose the correct `language` for the tradition color.
4. Set `generation` to one more than the musician's primary guru's generation (Tyagaraja = 0).

### Making UI or Logic Changes

Since everything is in `index.html`, edits to CSS go in the `<style>` block and JavaScript edits go in the `<script>` block. There is no module system or bundler.

### Coding Style

- **JavaScript**: Vanilla ES6, no frameworks. Uses arrow functions, `const`/`let`, template literals. Keep the same compact, minified CSS style already present. JavaScript can be more readable.
- **CSS**: Written minimized (space-separated rules on one line per selector) to keep file size down. Follow this style when adding CSS.
- **No external dependencies** should be added beyond D3.js (CDN) and Google Fonts. Keep the project self-contained.
- **Avoid introducing a build system.** The entire value of this project is that it's a single file anyone can open.

### Issue Tracking

Issues are stored in `localStorage` under the key `lineage-issues` as a JSON array. Each issue object has:
```js
{ id, timestamp, nodeName, nodeId, text }
```
Issues can be exported as JSON from the issues panel, and optionally submitted to the Google Apps Script endpoint.

## Testing

There is no automated test suite. Test changes by:
1. Opening `index.html` in a browser.
2. Verifying the graph renders without console errors.
3. Clicking nodes to confirm the detail panel populates correctly.
4. Testing search, filters, hover highlighting, and drag behavior.
5. Verifying new data entries appear in the graph with correct colors and connections.

For data changes, check:
- New nodes appear connected to correct parents via `guru_ids`.
- Node color matches the `language` field.
- The detail panel shows accurate information when the node is clicked.

## Git Workflow

```bash
# Standard flow
git add index.html
git commit -m "Descriptive message"
git push -u origin <branch-name>
```

Branch names follow the pattern `claude/<task-id>` for AI-assisted work.

## External Dependencies (CDN)

| Library          | Version | URL                                                                  |
|------------------|---------|----------------------------------------------------------------------|
| D3.js            | 7.8.5   | `https://cdnjs.cloudflare.com/ajax/libs/d3/7.8.5/d3.min.js`       |
| Playfair Display | latest  | Google Fonts                                                         |
| Source Sans 3    | latest  | Google Fonts                                                         |

No `package.json`, no `node_modules`, no lock files.
