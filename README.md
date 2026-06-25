# sql-content

A **content repo** for the `graphl-ux` learning app (sibling repo). It holds the
**SQL** concept — notebooks, per-section narration scripts, and the `manifest.json`
that wires sections to the scene — fetched by the app **at runtime** over raw GitHub.

There is **nothing to build, run, or test** here. Correctness is verified by the
`graphl-ux` app consuming this content. (The one executable is
`scripts/colab_generate_audio.ipynb` — a Colab tool that turns `tts/` scripts into
`audio/` `.wav`s.)

## Layout

```
manifest.json   # wires each module: notebook ref + per-section overlay (scene/spine/role/audio/highlight/focus)
notebooks/      # the teaching .ipynb (prose + code source of truth) — 01..08
tts/            # per-section narration scripts (plain spoken prose) — 55 files
audio/          # generated .wav narration (per-section) — generated on Colab (pending)
scenes/         # reserved/empty — the real scene lives in the graphl-ux app (src/scenes/sql.ts)
scripts/        # colab_generate_audio.ipynb — tts/*.tts -> audio/*.wav on a Colab GPU
DESIGN.md       # the FIXED visual house style (shared with apache-spark-content)
```

## The contract (shared with apache-spark-content)

1. **The notebook is the single source of truth** for a module's prose and code.
   The `manifest.json` only *wires* — it must never duplicate notebook content.
2. The app splits each notebook at every `## ` heading into **sections** (= pages),
   matched to the manifest overlay by **normalized heading text** (case / backticks /
   whitespace insensitive). The SQL notebooks number their headings (`## 1. …`), so the
   manifest `heading` keeps that `N. ` prefix verbatim. A heading edit must be mirrored.
3. A section's diagram **images are stripped** by the app — the **scene** replaces them.
4. **Scenes live in `graphl-ux`** (`src/scenes`), authored in TypeScript — not here.
   The SQL concept rides one map, `sql` (`src/scenes/sql.ts`). Here you only reference
   it **by id** in the manifest, and spotlight its `sql-*` node ids.

## The SQL scene (in graphl-ux, not here)

`sql` — one wide map ported from NodeMap: **DDL** (metadata mutators) ▸ **catalog**
(database/schema/table → columns/indexes/constraints/views) ▸ **storage**
(tablespace/pages/rows/WAL) ▸ the **query pipeline** in logical-execution order
(FROM ▸ JOIN ▸ WHERE ▸ GROUP BY ▸ HAVING ▸ SELECT ▸ DISTINCT ▸ ORDER BY ▸ LIMIT, with
CTE/window modifiers) ▸ **transactions** (BEGIN/COMMIT/ROLLBACK around the write DML,
plus isolation levels) ▸ **set operations**. Node ids are `sql-*` (e.g. `sql-where`,
`sql-join`, `sql-catalog`, `sql-constraints`, `sql-write-dml`, `sql-transactions`).
Highlighting a container id lights its children — spotlight `sql-join` to light all
six join types. The camera frames one clause / subsystem per section via `focus`.

## Narration (per-section TTS)

One `.tts` script **per section**, plain spoken prose — what a teacher would say at a
whiteboard. Naming: `tts/<NN>-<SS>-<slug>.tts` → `audio/<NN>-<SS>-<slug>.wav`, where
`NN` is the module number and `SS` the section order. The stem is shared by the
`.tts`, the `.wav`, and the manifest `audio` field. The notebook's "What's in this
notebook" overview and the trailing "Where to go next" outro are **not** narrated
(reading-only). See `apache-spark-content/CLAUDE.md` "TTS guidelines" for the full
style rules (plain prose, no markdown/code, spell out symbols — e.g. `%` → "percent
sign", `||` → "double pipe", SQL → "sequel", CTE → "common table expression").

## How content is served

The app fetches this repo at runtime over **raw GitHub**:
`https://raw.githubusercontent.com/schemabotview/sql-content/main/…`
A content change is live once pushed to `main` — no app rebuild needed.

## Source of notebooks

Notebooks are copied as-is from the runnable curriculum at `~/Projects/sql` (DuckDB).

## Status

- 8 notebooks copied in; **all 8 modules are scene-wired AND narrated** in
  `manifest.json` (every section → the `sql` scene with `spine`/`highlight`/`focus`/
  `role` + a per-section `audio` stem).
- **55 per-section `tts/*.tts` scripts** authored, 1:1 with the manifest `audio`
  stems. The notebook overview + outro are intentionally not narrated.
- The per-section `.wav`s still need a **Colab generation pass**
  (`scripts/colab_generate_audio.ipynb`, `tts/` → `audio/`). Until generated, a
  section's clip 404s in the app (auto-advance halts there) rather than narrating.
- The `sql` scene is authored + registered in graphl-ux (`src/scenes/sql.ts`,
  `src/scenes/index.ts`) and the concept is in the app catalog
  (`src/content/catalog.ts`). `npm run build` in graphl-ux passes.
- **Not yet pushed to GitHub** — once pushed to `main`, the live app pulls it.
