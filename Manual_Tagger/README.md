# Manual Tagger

A single-file, zero-dependency span annotation tool for named entity recognition — the
hand-labelling half of this repo, feeding the automated taggers alongside it.
Open `ner_annotator.html` in any modern browser and start tagging — no install, no
build step, no network access.

![labels](https://img.shields.io/badge/dependencies-none-success) ![offline](https://img.shields.io/badge/works-offline-success)

---

## Quick start

GitHub shows `.html` files as source code, so the tool does not run from the repo page.
Download or clone the repo first (**Code → Download ZIP**, or GitHub Desktop), then:

**Fastest** — double-click `ner_annotator.html`, then click **Open data…** and pick
`../data/tweets_c.json` from the repo's shared data folder.

**Better** — serve the repo so the tool can save straight back to your file and autosave
as you work. Run this from the **repo root** (the folder holding both `data/` and
`Manual_Tagger/`), so the shared data folder is reachable:

```bash
python -m http.server 8000
```

Then open <http://localhost:8000/Manual_Tagger/ner_annotator.html>. On Chrome or Edge
this unlocks **Save** writing in place to the file you opened, instead of dropping a copy
in Downloads. You can also skip the file picker entirely:

```
http://localhost:8000/Manual_Tagger/ner_annotator.html?data=../data/tweets_c.json
```

---

## Tagging

Select some text, then click a label or press its number key. That is the whole loop.

| Key | Action |
| --- | --- |
| <kbd>→</kbd> / <kbd>n</kbd> | Next record |
| <kbd>←</kbd> / <kbd>p</kbd> | Previous record |
| <kbd>1</kbd>…<kbd>9</kbd> | Tag the selection — or re-tag the selected span |
| <kbd>Delete</kbd> | Remove the selected span |
| <kbd>u</kbd> | Jump to the next record with no spans |
| <kbd>Ctrl</kbd>+<kbd>Z</kbd> | Undo (<kbd>Shift</kbd> to redo) |
| <kbd>Ctrl</kbd>+<kbd>S</kbd> | Save |
| <kbd>/</kbd> | Focus search |
| <kbd>?</kbd> | Shortcut list |

Click a highlighted span to select it. Fine-tune its boundaries with the start/end
number fields in **Spans on this record**. Sloppy selections are trimmed to the word —
grabbing `" Campo "` stores `Campo`.

Overlapping and nested spans are supported: the innermost one takes the colour and the
region is hatched, so `San Diego` can be `GPE` inside `San Diego Sheriff's Office` as
`ORG` without either one being lost.

---

## Your own label scheme

Eight defaults ship as quick picks, aligned with the OntoNotes types spaCy uses:

| Label | Covers |
| --- | --- |
| `GPE` | Countries, cities, states, counties |
| `LOC` | Non-GPE locations: regions, mountains, rivers, roads, water bodies |
| `FAC` | Buildings, airports, highways, bridges, piers, parks |
| `ORG` | Companies, agencies, institutions, teams |
| `PERSON` | People, including fictional |
| `NORP` | Nationalities, religious or political groups |
| `EVENT` | Named hurricanes, wildfires, battles, sports events |
| `HASH` | Hashtags and handles carrying entity meaning |

They are only a starting point. In **Manage** you can add, rename, recolour, describe
and delete labels freely:

- **Renaming** a label rewrites every existing span that uses it, after asking.
- **Deleting** one asks whether to drop its spans too or just retire it from the palette.
- **Descriptions** show on hover, which keeps a team consistent about what each tag means.
- Labels found in a loaded file are adopted automatically, so nothing in your data is
  ever unreachable.

The scheme lives in your browser and can be exported as a small file:

```json
{ "labels": [ { "name": "ROAD", "color": "#0891b2", "desc": "Named highways and streets" } ] }
```

Commit that next to your data and annotators import it with **Manage → Import scheme**,
so everyone tags against the same definitions.

---

## Data format

The native format is Doccano-compatible — the same shape as `../data/tweets_c.json`:

```json
[
  {
    "id": 9,
    "text": "Breaking: Fire crews are battling a brush fire burning north of Campo near I-8.",
    "Comments": ["north is loc"],
    "label": [[55, 60, "LOC"], [64, 69, "GPE"], [75, 78, "FAC"]]
  }
]
```

Offsets are character positions, end-exclusive: `text[start:end]` in Python and
`text.slice(start, end)` in JavaScript both give the span back.

It also reads, without being told which is which:

- **JSONL** — one record per line
- **Bundles** — `{"labels": [...], "examples": [...]}`
- **Span fields** named `label`, `entities`, `spans` or `annotations`
- **Span shapes** `[start, end, "LABEL"]` or `{"start":…, "end":…, "label":…}`
- **Text fields** named `text`, `content`, `sentence`, `tweet` or `body`

Fields it does not recognise are carried through untouched, so nothing in your file is
lost by opening it here.

### Exports

| Format | Use |
| --- | --- |
| JSON | Same schema as the input — the round trip is lossless |
| JSONL | Line-delimited, for streaming pipelines |
| Bundle | Labels + records together, to hand a task to someone |
| CoNLL / BIO | `token TAG` per line, ready for sequence labelling |
| Label scheme | Just the tag set |

---

## Not losing work

Every change is autosaved to this browser within a second, and the tool offers to
restore it next time you open the page. `Save` writes a real file: in place on
Chrome/Edge over `http(s)`, otherwise as a download.

Autosave needs browser storage, which is blocked on some `file://` setups. When that
happens the tool says so in a banner rather than pretending to save — serve the folder
over `http` as above and it works.

---

## Data health

The sidebar scans the loaded dataset continuously and flags, with one-click fixes:

- duplicate `id` values
- duplicate record text
- spans that fall outside their text, or are empty
- spans starting or ending on whitespace
- records with overlapping spans

`../data/tweets_c.json` currently reports **7,336 records, 6,678 labelled**, and two real
problems worth knowing about:

- **ids are not unique** — 7,336 records carry only 4,025 distinct ids, one appearing
  three times. It looks like several exports were concatenated. Use *Renumber ids 1…N*
  before keying anything on `id`.
- **249 spans start or end on whitespace**, which will shift your token alignment. Use
  *Trim whitespace in spans*.

---

## Tests

From the repo root:

```bash
python -m http.server 8000
```

then open <http://localhost:8000/Manual_Tagger/tests/test_annotator.html>. It loads the real tool in a
frame and drives it: parsing, span arithmetic, overlap rendering, live-DOM offset
mapping, editing, undo, navigation, filtering, export round trips, real records from
`tweets_c.json`, and a 7,336-record scale check. 65 assertions, all currently passing.

The suite must be served over `http` — browsers block cross-frame access between
`file://` pages, and the page tells you so rather than reporting false failures.

---

## Why there is no framework

The previous version loaded React, ReactDOM, Babel and Tailwind from CDNs and compiled
JSX in the browser on every page load. This one is plain HTML, CSS and JavaScript in one
file. It starts instantly, runs with the network off, and cannot be broken a year from
now by a CDN moving on. Loading and rendering all 7,336 tweets takes about 5 ms; a full
JSON export takes about 3 ms.
