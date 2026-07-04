# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**SonicNote Sync** — Obsidian plugin (id: `sonicnote-sync`) that syncs recordings from the SonicNote (妙记) cloud service into an Obsidian vault as Markdown files.

Each synced recording becomes a single `.md` file with YAML frontmatter metadata, plus optional sections for notes, AI summary, study report, and full transcript with speaker diarization.

## Build & Dev Commands

```bash
npm run dev        # Watch mode — rebuilds main.js on source change
npm run build      # Production build — type-check then minify via esbuild
```

The entry point is `src/main.ts`. esbuild bundles it to `main.js` in CommonJS format (target ES2018), externalizing `obsidian`, `electron`, and `@codemirror/*`.

To install in Obsidian for testing:
- Symlink or copy `main.js`, `manifest.json`, `styles.css` into `<vault>/.obsidian/plugins/sonicnote-sync/`
- Enable the plugin in Obsidian → Settings → Community Plugins

## Architecture

### Dependency Direction

```
main.ts (Plugin lifecycle)
  ├── api.ts (HTTP client — no plugin deps)
  ├── sync.ts (Sync orchestration — depends on api + formatter)
  ├── settings.ts (Settings UI tab + login modal — depends on api)
  ├── types.ts (Shared types, zero deps)
  └── formatter.ts (Pure Markdown generation, zero deps)
```

### Key Modules

- **`main.ts`** — Plugin entry. Registers ribbon icon, 3 commands (sync/login/logout), status bar item. Manages auto-sync timer. Wires everything together: loads settings → creates `SonicNoteApiClient` → creates `SyncService` → creates `SonicNoteSettingTab`.

- **`api.ts` — `SonicNoteApiClient`** — Wraps all HTTP calls to the SonicNote backend (`/app/recording/list`, `/app/recording/detail`, `/app/recording/getNote`, `/share/{audioId}/transcript/result`, `/share/{audioId}/summary`, `/share/{audioId}/studyReport`, `/app/mcp/login`). Auth is Bearer token from settings. Uses Obsidian's `requestUrl` (not `fetch`), which is required for mobile/CORS compatibility.

- **`sync.ts` — `SyncService`** — The core orchestration logic. `syncAll()`:
  1. Ensures the target folder exists in the vault
  2. Builds a local index by scanning existing `.md` files and extracting `audio_id` from their frontmatter
  3. Paginates through ALL recordings from the backend API
  4. For each recording: skips if already synced (unless the filename matches the original name but server has a new title — then it renames), otherwise fetches transcript/summary/study-report/note **in parallel** and writes a new Markdown file
  5. Updates `lastSyncTime` in plugin settings

- **`formatter.ts`** — Pure functions for generating Markdown output. No side effects. `toMarkdown()` orchestrates the full file content: frontmatter → title → note → AI summary → study report → transcript. `formatTranscript()` converts `[{spokesperson, text, time}]` to speaker-labeled timestamped lines.

- **`settings.ts`** — Obsidian settings tab with sections for: sync folder, transcript toggle, auto-sync, resync interval dropdown, frontmatter field toggles (builtin + custom), and login/logout with a `LoginModal`.

- **`types.ts`** — All TypeScript interfaces: `SonicNotePluginSettings` (with `DEFAULT_SETTINGS`), `Recording`, `TranscriptSegment`, `SummaryData`, `StudyReportData`, `BackendResponse`, `SyncResult`, `LocalFileInfo`, and `BUILTIN_FRONTMATTER_FIELDS` (10 metadata fields map).

### Data Flow

```
User clicks sync / auto-sync timer fires
  → main.ts triggerSync()
    → SyncService.syncAll()
      → ApiClient.fetchRecordingList() [paginated loop]
      → for each recording:
          → ApiClient.fetchTranscriptResult() ┐
          → ApiClient.fetchSummary()          ├ parallel
          → ApiClient.fetchStudyReport()      │
          → ApiClient.fetchNote()             ┘
          → formatter.toMarkdown()
          → app.vault.create() or app.vault.modify()
      → saveSettings() with lastSyncTime
```

### Sync Deduplication

The local index is a `Map<audioId, LocalFileInfo>` built by scanning frontmatter of existing `.md` files in the sync folder. A recording is skipped if its `audioId` already exists in the index, **unless** the file still uses the original filename but the server now has a `recordNickName` — in that case the file is renamed to match the new title.

### Skipped Plugin Files

The following are in `.gitignore` and never committed:
- `node_modules/`
- `main.js` and `main.js.map` (build artifacts)
- `data.json` (Obsidian plugin settings stored in the plugin folder)

### Skills Directory

The `skill/` folder contains Claude Code skill definitions:
- `md-todo-extractor/` — Extract todo items from Markdown
- `work-summary/` — Generate work summaries

The `release/` folder contains packaged `.zip` files for Obsidian plugin distribution and skill creation prompts (not part of the plugin build).
