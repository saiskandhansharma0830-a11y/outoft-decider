# Outfit Decider

> **Read this first: what changed since the original v0 write-up.** The sections further down describe v0 in detail and are still mostly accurate; this block overrides them where they differ.
>
> - **Now an installable phone app (PWA).** Files: `index.html` (all app logic), `manifest.webmanifest`, `sw.js` (offline cache; bump `CACHE_VERSION` whenever any cached file changes), `icons/` (generated PNGs). It must be served over HTTPS (or `localhost`) to install; opening `index.html` as a plain file still works but cannot be installed or work offline.
> - **Schema is now v4, engine `rules-5.0`.** Laundry added (see below). Older data upgrades automatically (`MIGRATIONS[3]`). New localStorage key `outfitDecider.wears` (append-only laundry log).
> - **Laundry system.** Every item has `washAfter` (1–30 wears, or `null` = never tracked), `wearsSinceWash`, `lastWornAt`, `lastWashedAt`. The Decide card has **I wore this** (once per outfit, with Undo). An item is *dirty* when `wearsSinceWash >= washAfter` and is a hard exclusion in `rankOutfits` (`activeWardrobe()`). Closet shows an "In the wash" group with **Mark washed / Wash all / To wash**, and a limit selector per item. If nothing clean is left for a category, Decide lists the dirty items with inline **Mark washed** buttons. Every wear/undo/wash/send-to-wash is a `laundry_event` in the wear log (included in backups). Feedback/history snapshots carry `wearsBefore` and `washAfter`.
> - **Colours are no longer limited to 18 names.** 45 curated shades (grouped in the picker) plus a **custom colour** picker. An item's `color` is either a preset id (`"sage"`) or a custom hex (`"#7a8b6c"`), so no data migration was needed. The 18 original colours keep their hand-authored engine values (verified: 1,704 reference scores unchanged); the 27 new presets and every custom colour get their attributes from `deriveAttrs(hex)` (OKLab/OKLCH based; hue mapped to the red-yellow-blue wheel). Custom colours are named after the nearest preset in the same family. Engine special cases now use derived flags (`isBlack`, `isDenim`, "same shade" = OKLab distance < 0.04) instead of colour names.
> - **Safe import.** `importBackup` parses, migrates and validates first, shows exactly what will be restored, keeps a `outfitDecider.preImport.<timestamp>` safety copy of current data, and never silently drops anything. A first-run "Import a backup" card shows on My Closet while the app is empty. Backups also carry the wear log.
> - **My Data additions:** Install card (Android install button / iPhone instructions), Share backup (phone share sheet), last-backup status, a backup reminder on Decide, storage-protection status (`navigator.storage.persist()`).
>
> ### Moving from the old HTML file to the installed app
> Storage is per address, so the installed app starts empty.
> 1. In the **old** copy (same browser, same way you always opened it): My Data → **Download backup**; also Export JSONL as a second copy. Keep the files safe. Do not clear browser data and do not delete the old file yet.
> 2. Install the new app (below), open it **from its home-screen icon**, and on My Closet choose **Import a backup** (or My Data → Import backup). Check the item and feedback counts match.
> 3. On iPhone, the home-screen app and Safari keep **separate** data: import inside the installed app, not in Safari.
> 4. There is no sync between devices. Back up regularly.
>
> ### Hosting for the phone (free, needs your go-ahead)
> Recommended: **GitHub Pages** (public repo; it only exposes app code, never your data). Create a repo, push `index.html`, `manifest.webmanifest`, `sw.js`, `icons/`, then Settings → Pages → deploy from `main` / root. Alternative without git: **Netlify Drop** (drag the folder onto app.netlify.com/drop). Then open the URL on the phone: Android Chrome → Install; iPhone Safari → Share → Add to Home Screen.
> To test locally with real storage and a service worker: `npx serve .` and open the printed `http://localhost:…` address.
>
> ### Not verified here
> The service worker and install flow could not be run in the development browser. `sw.js` logic was checked against a mock; offline use, install, Lighthouse "installable" and real-phone behaviour must be verified on the hosted URL.

# Outfit Decider v0 (original write-up)

A personal wardrobe assistant. You list the clothes you own, tell it the weather and occasion, and it recommends **one well-coordinated outfit at a time** using only your closet. It explains why the outfit works, lets you reroll, and records your Good/Bad feedback as a structured dataset for a **future** personalised AI system.

> **Important:** the recommendation engine is deterministic JavaScript (colour-theory rules). There is **no AI model** and nothing is being trained. Feedback is only *collected*; it deliberately does not change recommendations (see [Design decisions](#design-decisions)).

---

## 1. Quick start

- The whole app is one file: `index.html`. No build step, no dependencies, no backend, no network.
- Open it by double-clicking, or in VS Code use the **Live Server** extension (recommended, because `localStorage` behaves like a real site that way) or run:

```powershell
npx serve .
```

- Data lives in the browser's `localStorage`, **per browser and per origin**. A `file://` page and `http://localhost:xxxx` are different origins with separate data. Use **My Data → Download backup** to move data between them.
- Syntax-check the script block without a browser:

```powershell
# PowerShell: extract the <script> body and check it with Node
$html = Get-Content index.html -Raw
$js = [regex]::Match($html, '(?s)<script>(.*)</script>').Groups[1].Value
Set-Content "$env:TEMP\od.js" $js
node --check "$env:TEMP\od.js"
```

## 2. Project layout

```
mahFIT/
  index.html   <- the entire app (HTML + CSS + JS), ~1,640 lines
  README.md    <- this file
```

`index.html` order: `<style>` (design tokens, components) → markup (header, three sections) → one `<script>` block, organised in numbered sections:

| Section | What is in it | Approx. lines |
|---|---|---|
| 1. Constants & reference data | versions, storage keys, `TYPES`, `WEATHER`, `OCCASIONS`, `COLORS` (colour metadata) | 299–385 |
| 2. Utilities | `$`, `$$`, `uid`, `clamp`, `toast`, `colorOf`, intensity | 391–413 |
| 3. Storage layer | `Store` (read/write/remove), `MIGRATIONS`, `migrateData`, `runMigrations` | 418–541 |
| 4. App state | `state`, `sanitizeItem`, `withTraits`, `loadState`, save helpers | 546–595 |
| 5a. Colour-theory engine | `colourRelation`, `analyseColours`, tiers | 601–874 |
| 5b. Context and ranking | `contextAdjustments`, `scoreOutfit`, `decideHoodie`, `rankOutfits`, reroll selection, `explain` | 877–1078 |
| 6. Generation and feedback | `generate`, snapshots, `buildFeedbackEvent`, `recordFeedback` | 1081–1238 |
| 7. Rendering | Decide card, empty/missing/no-good-outfit states, closet UI, recent outfits, My Data | 1240–1490 |
| 9. Export / import / clear | `exportJSONL`, `exportCSV`, `exportAll`, `importBackup`, `clearKey` | 1493–1604 |
| 10. Boot | `renderAll`, `init` | 1606–end |

(Line numbers drift as you edit; search for the function names.)

## 3. Product behaviour

### Navigation
Two primary sections via a segmented control: **My Closet** and **Decide**. **My Data** is a text link in the header (kept out of the main navigation on purpose). On load the app opens **Decide** if the closet has items, otherwise **My Closet**.

### My Closet
- Add an item with exactly three fields: **name** (required, trimmed, whitespace-only rejected, max 60 chars), **type**, **colour**.
- Types: T-shirt, Shirt, Hoodie, Pants, Jeans, Shoes.
- Colours (18): White, Black, Grey, Navy, Blue, Light blue, Beige, Cream, Brown, Olive, Green, Burgundy, Red, Pink, Yellow, Orange, Purple, Denim.
- Items get a unique ID (`itm_…`); duplicate names are allowed.
- Delete is two-tap ("Delete" → "Tap to confirm", resets after 4 s). Deleting never touches history or feedback (they hold snapshots).

### Decide
1. Pick **weather** (Hot/Warm/Mild/Cool/Cold) and **occasion** (Casual/Smart casual/Smart/Active). Both persist.
2. **Decide my outfit** shows exactly one outfit: Main top, optional Hoodie layer, Bottom, Shoes (each with name, type, colour), a tier + score out of 100, and **Why this works**.
3. **Good / Bad / Reroll** sit directly under the outfit.
4. Outfit structure is fixed: exactly one top (T-shirt or Shirt), optional hoodie, one bottom (Pants or Jeans), one pair of shoes. Shoes are **required**.

### The four situations
| Situation | Behaviour |
|---|---|
| A. Strong outfits exist | Show the best one. |
| B. Only Acceptable/Weak exist | Show the best, with a "your options are limited" banner. |
| C. Every complete outfit clashes | Show "No harmonious outfit available" with the main problems. Nothing is recommended. |
| D. A required category is missing | Name what is missing (top / bottom / shoes). No partial outfit. |

### Reroll
- Candidate pool = outfits within `REROLL_WINDOW` (15) points of the best and never below Acceptable when something Acceptable-or-better exists.
- Prefers an unseen **visible look** (hoodie-or-top + bottom), then avoids the five recent outfits, then any unseen variation, then starts a new cycle with a notice.
- If only one outfit meets the bar: "No meaningful alternative…" (nothing is added to history).

### Recent outfits
The last **5** generated outfits (`HISTORY_LIMIT`) appear under Decide with colour dots, date, context and rating badge.

## 4. The recommendation engine

All code is in sections 5a/5b. It is pure rules, fully deterministic (ties broken by outfit key).

### 4.1 Colour metadata (`COLORS`)
Each colour has `hex`, `family`, `hue` (artist's RYB wheel, red 0 / orange 60 / yellow 120 / green 180 / blue 240 / purple 300; `null` for achromatic), `temp` (warm/cool/neutral), `lightness`, `saturation`, `neutrality` (0–1), `role` (foundation / versatile / muted / soft / accent). A derived `intensity` (how loud it looks) is computed at load. All values are **approximations of a typical garment** in that colour; the app only knows colour names, not real shades. Navy, brown and denim are treated as near-neutrals.

### 4.2 Pair relations (`colourRelation`)
Blends a *neutral lens* (`neutralPairScore`) and a *colour-wheel lens* (`chromaticPairScore`) weighted by how neutral the more neutral colour is. Score range −1…1. It recognises: neutral pairings, tonal/monochrome (flat if too close in lightness), analogous, awkward-gap (discordant), triadic, complementary (good only if one colour is soft or lightness differs a lot; both loud = vibrates), pastel pairs, warm/cool temperature handling, and special cases (black on black, denim on denim, navy + black "near-miss").

### 4.3 Whole-outfit analysis (`analyseColours`)
Evaluates **all visible pieces**, weighted by visual area (a hoodie hides most of the top: hoodie 0.40 / top 0.12 / bottom 0.38 / shoes 0.10 when worn). Colour score out of 100:

| Component | Max points |
|---|---|
| Harmony (all visible pairs) | 35 |
| Saturation balance (one focal colour, quiet support) | 20 |
| Light/dark contrast (upper vs bottom) | 15 |
| Temperature (warm/cool bridged or intentional) | 10 |
| Shoes (finish / echo another piece) | 10 |
| Visual balance (number of non-neutral colour families) | 10 |

Each component value in −1…1 is mapped with `(v + 0.25) / 1.25` into 0…max.

It also returns human-readable `notes`, a `clash` string if rejected, and structured `details` (relationships, loud colours, temperature verdicts, lightness gap) that go into feedback records.

### 4.4 Tiers and rejection
`TIERS`: Excellent ≥ 85, Good ≥ 74, Acceptable ≥ 62, Weak ≥ 45, otherwise Clashing. An outfit is **rejected** (never shown) if any pair scores ≤ −0.6 (serious clash), two unrelated loud colours appear, or the colour score is below 45.

### 4.5 Context and total (`contextAdjustments`, `scoreOutfit`)
Formality (from type vs occasion; being under-dressed costs more than over-dressed) and weather (warmth of the top, hoodie suitability) add or subtract points. `rankScore = colour + formality + weather`; the displayed `total` is that clamped to 0–100. Types carry fixed traits: e.g. T-shirt formality 0 warmth 0, Shirt 2/1, Pants 1.5/1, Jeans 0.5/1, Hoodie 0/2.

### 4.6 Hoodie decision (`decideHoodie`)
The hoodie is an optional layer of the *same* outfit. For every top+bottom+shoes combination the engine scores the complete outfit **without** a hoodie and **with each** hoodie, then:
- Hard blocks: hot weather, or Smart occasion.
- A hoodie is added only if it improves the overall score by more than `HOODIE_MIN_GAIN` (1.5) **and** does not lower the colour score by more than `HOODIE_MAX_COLOUR_LOSS` (3). A hoodie that would clash is never used.
- Produces a specific note ("X added: it lifts the colour harmony from 79 to 89; …" or "No hoodie: …").

### 4.7 Ranking and selection
`rankOutfits` builds every combination, drops rejected ones, and sorts by `rankScore` (ties by key). `generate()` shows the top outfit on a fresh decision; rerolls go through `nextFromSession` as described above.

## 5. Data model and storage

### localStorage keys
| Key | Contents |
|---|---|
| `outfitDecider.meta` | `{ schemaVersion, createdAt, … }` |
| `outfitDecider.wardrobe` | array of items |
| `outfitDecider.history` | last 5 generated outfits |
| `outfitDecider.feedback` | **append-only** array of feedback events |
| `outfitDecider.prefs` | `{ weather, occasion }` |
| `…feedback.unreadable.<ts>` / `…unexpected.<ts>` | automatic backups of data that could not be parsed or had a wrong shape |

`SCHEMA_VERSION = 3`, `ENGINE_VERSION = 'rules-4.0'`.

### Wardrobe item (schema v2+)
```json
{ "id": "itm_…", "name": "Navy tee", "type": "tshirt", "color": "navy", "createdAt": "ISO-8601" }
```
Formality/warmth/role are derived from `type` via `withTraits()`, not stored.

### History entry
`id` (`out_…`), `createdAt`, `context`, `itemIds`, `snapshot` (full garment snapshots), `score`, `tier`, `rank`, `candidates`, `rerollIndex`, `outfitKey`, `rating`, `feedbackId`.

### Feedback event (schema v3), one per Good/Bad tap
```
recordType: "outfit_feedback"
schemaVersion, engineVersion
eventId (fbk_…), outfitId (out_…), timestamp (ISO-8601), label ("good" | "bad")
supersedesEventId        <- previous event for this outfit if the user changed their mind
context: weather, occasion, outfitGeneratedAt, rank, candidates, eligibleCandidates,
         rejectedCandidates, rerollIndex, wardrobeSize
outfit:  hoodieIncluded, top, hoodie|null, bottom, shoes   (full garment snapshots incl. colour metadata)
recommendation: overallScore, tier, colourScore, colourTier, components, contextAdjustments,
         saturation{…}, temperature{…}, contrast{…}, colourRelationships[…],
         hoodie{ included, improvedOutfit, hoodiesConsidered, scoreDelta, colourGain, withoutHoodie, options[], note },
         explanation{ strengths[], cautions[] }
```
Events are **never edited or deleted** except by the explicit "Delete feedback" / "Reset everything" buttons. The "current" label for an outfit is its latest event (`latestFeedbackFor`, `currentLabels`). Old v2 feedback is converted by `MIGRATIONS[2]` with `legacy: true` and original reasons kept.

### Migrations
`MIGRATIONS[n]` upgrades data from version n to n+1; `migrateData` chains them and is also used when importing a backup. To change the schema: bump `SCHEMA_VERSION`, add `MIGRATIONS[oldVersion]`, keep old feedback readable.

### Storage safety (`Store`)
- Probes `localStorage`; falls back to in-memory and shows a banner if unavailable.
- Unparseable data is copied to a `.unreadable.<ts>` key, then the original key is cleared, and a banner explains.
- Write failures (e.g. quota) return `false`; callers roll back (feedback is **not** shown as saved if the write fails).

## 6. Exports

All on **My Data**:
- **Export JSONL** → `outfit-feedback-YYYY-MM-DD.jsonl`: one complete event per line, UTF-8.
- **Export CSV** → `outfit-feedback-YYYY-MM-DD.csv`: one row per event, flattened (76 columns: ids, timestamp, label, `is_current_label`, context, scores per component, saturation/temperature verdicts, relationships text, hoodie fields, and `top_*`/`hoodie_*`/`bottom_*`/`shoes_*` columns). UTF-8 BOM and CRLF for Excel; commas/quotes/newlines escaped; text starting with `= + - @` gets a leading `'` to stop formula injection; missing hoodie columns are blank and `hoodie_included=false`.
- **Download backup / Import backup**: full JSON (wardrobe, history, feedback, prefs); import migrates old versions and replaces current data after a confirm.
- Danger zone: Clear recent outfits, Delete feedback, Reset everything (all confirm first).

Python example:
```python
import json, pandas as pd
rows = [json.loads(l) for l in open("outfit-feedback-2026-11-18.jsonl", encoding="utf-8")]
df = pd.json_normalize(rows)
```

## 7. Design decisions (and why)

- **Single file, no frameworks:** original requirement; keep it that way unless you decide to split.
- **Feedback does not affect scoring:** Phase 5 requires raw data collection kept separate from the engine. An earlier version nudged scores from ratings; that was removed on purpose.
- **Append-only feedback:** changing Good→Bad adds a new event instead of overwriting, so the dataset keeps the full history of changes.
- **Snapshots everywhere:** history and feedback copy garment details, so deleting/editing a closet item cannot damage past records.
- **Shoes required:** a "complete outfit" always has shoes.
- **Reject, don't force:** clashing outfits are never shown just to have an answer.
- **Hoodie area weighting:** a hoodie hides most of the top, so top-vs-bottom harmony counts less once a hoodie is on.
- **Design:** white background, system font stack, one blue accent, no emoji, no heavy animation (`prefers-reduced-motion` respected), 16 px inputs (no iOS zoom), 44 px+ tap targets.

## 8. Testing

There is no automated test suite; testing was done interactively in a browser by calling the app's own functions from the console. Useful console snippets:

```js
// seed a closet quickly
const mk = a => a.map(([n,t,c]) => sanitizeItem({name:n,type:t,color:c}));
state.wardrobe = mk([['White tee','tshirt','white'],['Denim jeans','jeans','denim'],
                     ['White sneakers','shoes','white'],['Navy hoodie','hoodie','navy']]);
state.prefs.weather = 'cool'; generate(false);

colourRelation('red','green')           // pair score + reason
scoreOutfit(outfit, {weather:'mild',occasion:'casual'}, null)
rankOutfits({weather:'mild',occasion:'casual'})   // {ranked, rejected, topClashes}
state.feedback.map(validateFeedbackEvent)         // all null = valid
```

Scenarios verified for v0 (all passed): empty wardrobe; missing top/bottom/shoes; no hoodies; several hoodies; hoodie helps / hoodie hurts; one suitable combination; many combinations; all combinations clash; Good, Bad, changing feedback; persistence across reload; deletion of items keeping snapshots; valid ISO timestamps and unique IDs; JSONL/CSV validity and escaping; empty export; missing/malformed/full/unavailable storage.

**Not verified:** a physical phone, Safari/Firefox, and very large datasets. Worth automating next (Playwright would suit a single-file app).

## 9. Known limitations

- Colours are labels only, so "Blue" is a typical blue; no real shade, pattern, fabric or fit.
- Formality and warmth come from type alone (a "Shirt" is always formal-ish).
- Data is per-browser; no sync. Clearing site data deletes it unless backed up.
- Only the last 5 outfits are kept in history (feedback keeps its own records).
- No learning from feedback yet (by design).
- Single hoodie layer only; no jackets, accessories or multiple tops.

## 10. Ideas for future work

1. Analyse the exported dataset (Python) to see which pairs people rate Bad, and tune `colourRelation` weights by hand or fit a model offline.
2. Optional "personalisation" layer that reads feedback *outside* the core engine (keep it separable and switchable).
3. Let users mark an item as "in the laundry" (removed in Phase 2; the engine would only need to filter `state.wardrobe`).
4. Split `index.html` into modules (colour data, engine, storage, UI) with a bundler, and add unit tests for `colourRelation`/`analyseColours` and the migrations.
5. More item types (jackets, shorts, skirts), patterns, and per-item shade choice.
6. A feedback reason picker (it existed in an earlier version and was removed to keep rating one tap).
7. PWA manifest + service worker for installable offline use.

## 11. Editing tips for VS Code

- Search by function name (`Ctrl+Shift+F`), e.g. `analyseColours`, `decideHoodie`, `buildFeedbackEvent`.
- When you change the shape of stored data, bump `SCHEMA_VERSION` and add a migration.
- When you change scoring, bump `ENGINE_VERSION` so exported feedback records show which engine produced them.
- Beware of `$` vs `$$`: `$` returns one element, `$$` returns an array (a mix-up once broke tab switching).
- If you script edits with `String.replace`, use a replacement *function* (`s.replace(a, () => b)`), because `$$` in replacement strings is interpreted specially.
- `localStorage` is shared per origin: if testing looks "stuck", clear it (`localStorage.clear()` in DevTools) and reload.
