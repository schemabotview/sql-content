# CLAUDE.md — sql-content

A **content repo** (not an app) for the `graphl-ux` engine. It holds the **SQL**
concept: notebooks, per-section narration, and the `manifest.json` that wires them.
The app fetches this repo's `manifest.json` + notebooks over the network and renders
them. Read `README.md` first; this file is the working orientation.

This repo is a sibling of `apache-spark-content` / `java-content` and follows the
**same contract** — when in doubt, mirror what those repos do (apache-spark-content's
`CLAUDE.md`/`DESIGN.md` are the mature reference, especially the **TTS guidelines**
and the **fixed semantic palette**).

## The core contract (do not break)

1. **The notebook is the single source of truth** for prose and code. `manifest.json`
   only *wires* sections — never duplicate notebook content here.
2. The app splits each notebook at every `## ` heading into sections (= pages) and
   matches them to the manifest by **normalized heading** (lowercase, backticks
   stripped, whitespace collapsed — see graphl-ux `content/module.ts`). The SQL
   notebooks number their headings (`## 1. What SQL is`), and the normalizer does
   **not** strip that `N. ` prefix — so the manifest `heading` must keep it verbatim.
3. Section diagram **images are stripped** by the app — a **scene** replaces them.
4. **Scenes live in `graphl-ux`** (`src/scenes`), authored in TypeScript — not here.
   The `scenes/` dir is reserved/empty. Reference the scene by **id** only.

## The SQL scene (in graphl-ux, not here)

One dense map, ported from NodeMap's `src/data/scenes/sql.ts` into a `SceneSpec`,
registered in `graphl-ux/src/scenes/index.ts` as **`sql`** (`src/scenes/sql.ts`).

The layout is the lesson: **DDL** mutates the **catalog** (metadata), the catalog
describes **storage** (data), the **query pipeline** scans storage in logical-execution
order, **set ops** compose pipeline outputs, and the **transaction** box wraps the
write DML that closes the loop back to storage. Node ids are `sql-*`:

- DDL: `sql-create-table`, `sql-alter-table`, `sql-drop-table`, `sql-truncate`,
  `sql-create-index`, `sql-create-view`, `sql-rename` (container `sql-ddl`).
- Catalog: `sql-database`, `sql-schema`, `sql-table` ⊃ `sql-columns`, `sql-indexes`,
  `sql-constraints` (⊃ `sql-pk`/`sql-fk`/`sql-not-null`/`sql-unique`/`sql-check`),
  `sql-views` (container `sql-catalog`).
- Storage: `sql-tablespace`, `sql-pages`, `sql-rows`, `sql-wal` (container `sql-storage`).
- Pipeline: `sql-from`, `sql-join` (⊃ `sql-inner-join`/`sql-left-join`/`sql-right-join`/
  `sql-full-outer-join`/`sql-cross-join`/`sql-self-join`), `sql-where`, `sql-group-by`,
  `sql-having`, `sql-select`, `sql-distinct`, `sql-order-by`, `sql-limit`, plus
  modifiers `sql-cte`, `sql-window` (container `sql-pipeline`).
- Transactions: `sql-begin`, `sql-write-dml` (⊃ `sql-insert`/`sql-update`/`sql-delete`/
  `sql-merge`), `sql-commit`, `sql-rollback`, `sql-isolation` (⊃ the four levels)
  (container `sql-transactions`).
- Set ops: `sql-union`, `sql-union-all`, `sql-intersect`, `sql-except` (container `sql-set-ops`).

Highlighting a container id lights its children, so spotlight the box (`sql-join`) to
light a whole group. Always check this list before adding a `highlight`/`focus`.

## manifest.json shape

Top level: `concept`, `design`, `scenes[]` (id/title/status), `presentations[]` (one
per module). Each presentation: `id`, `title`, `notebook`, `defaultScene` (`"sql"`),
`sections[]`. Each section overlay: `heading` (must match a notebook `## ` heading,
normalized — keep the `N. ` number), `scene` (`"sql"`), `spine` (bool — feed-mode
narration), optional `role` (`"hook"` on the first section), optional `audio`
(repo-relative `.wav`), optional `highlight` (scene node ids — AMBER, rest dim),
optional `focus` (node id(s) the camera frames).

## Narration

One `.tts` per section, plain spoken prose. Naming `tts/<NN>-<SS>-<slug>.tts` →
`audio/<NN>-<SS>-<slug>.wav`; the stem is shared by the `.tts`, the `.wav`, and the
manifest `audio` field. **Drop the notebook's "What's in this notebook" overview and
the trailing "Where to go next" outro** — those are reading-only, not narrated. Follow
`apache-spark-content/CLAUDE.md` "TTS guidelines" and this repo's source notebook
`~/Projects/sql/CLAUDE.md` (SQL-specific symbol spell-outs: `%` → "percent sign",
`||` → "double pipe", `COUNT(*)` → "count star", SQL → "sequel", CTE → "common table
expression", PK/FK → "primary key"/"foreign key").

The source curriculum at `~/Projects/sql/tts/` has **one `.tts` per notebook** (a
monologue). The per-section files here were authored fresh from each notebook's `## `
sections — one short script per section, anchored to what's on screen.

## When wiring a module

1. Confirm the notebook is in `notebooks/` and its `## ` headings are final.
2. Add a `presentations[]` entry; list every numbered section (each becomes a page),
   `heading` matching the notebook verbatim (keep the `N. ` prefix).
3. Every section uses `scene: "sql"`; set `spine` for essentials, `role: "hook"` on
   the first, and `highlight`/`focus` to the relevant `sql-*` ids.
4. Author one `tts/<NN>-<SS>-<slug>.tts` per section, set each `audio`, then run
   `scripts/colab_generate_audio.ipynb` to generate + push the `.wav`s.

## Status

- 8 notebooks (DuckDB curriculum, `~/Projects/sql`) copied in.
- **All 8 modules scene-wired AND narrated** in `manifest.json` — every section →
  `sql` with `spine`/`highlight`/`focus`/`role` + a per-section `audio` stem.
- **55 `tts/*.tts` scripts**, 1:1 with the manifest `audio` stems (notebook overview +
  outro intentionally not narrated). `.wav`s **pending a Colab pass**.
- The `sql` scene is authored + registered in graphl-ux and the concept is in the app
  catalog (`catalog.ts`, accent TEAL, `contentBaseUrl` → this repo on raw GitHub).
  `npm run build` in graphl-ux passes.
- **Not yet pushed to GitHub.** Once pushed to `main`, the live app pulls it (the
  catalog already points the `sql` concept here).
