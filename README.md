# Geospatial NER

Tools for finding geospatial named entities — places, facilities, regions, events — in
short social-media text. The repo holds both halves of that job: a **manual tagger** for
building and correcting gold-standard annotations by hand, and **automated taggers** that
predict them.

```
Geospatial_NER/
├── data/                  shared corpus, read by every tool here
│   └── tweets_c.json      7,336 tweets, 6,678 hand-labelled
├── Manual_Tagger/         browser-based span annotation tool
│   ├── ner_annotator.html the tool — one file, no dependencies
│   ├── README.md          usage, data format, label schemes
│   └── tests/             65-assertion suite driving the real tool
└── LICENSE
```

## The corpus

`data/tweets_c.json` is the shared source of truth. Records are Doccano-style, with
entity spans as end-exclusive character offsets:

```json
{
  "id": 9,
  "text": "Breaking: Fire crews are battling a brush fire burning north of Campo near I-8.",
  "Comments": ["north is loc"],
  "label": [[55, 60, "LOC"], [64, 69, "GPE"], [75, 78, "FAC"]]
}
```

7,336 records, 6,678 of them carrying at least one span, across eight entity types (the
catalog also defines `ADD` for street addresses, not yet used in this corpus):

| Label | Spans | | Label | Spans |
| --- | ---: | --- | --- | ---: |
| `GPE` | 4,262 | | `HASH` | 2,089 |
| `ORG` | 4,215 | | `LOC` | 1,311 |
| `EVENT` | 2,598 | | `FAC` | 889 |
| `PERSON` | 2,585 | | `NORP` | 342 |

Two known defects, both flagged and fixable inside the manual tagger:

- **ids are not unique** — 7,336 records share only 4,025 distinct ids, one appearing
  three times, so the file looks like several exports concatenated. Do not key on `id`
  without renumbering first.
- **249 spans begin or end on whitespace**, which will shift token alignment in training.

No record contains characters outside the Basic Multilingual Plane, so Python character
offsets and JavaScript UTF-16 offsets agree throughout — no emoji-induced drift.

## Manual tagger

A single self-contained HTML file: no framework, no build step, no network. Download or
clone the repo and double-click `Manual_Tagger/ner_annotator.html` — GitHub shows `.html`
as source, so it will not run from the repo page itself.

Select text, press a label key, move on. Everything is autosaved as you go.

Labels are organised into **tag sets** — named palettes you switch between. Three ship
with the tool (`Core entities`; `Geospatial + social`; `Street addresses`, which adds
`ADD` for exact street addresses), and you can edit them or build your own; sets export
as one file so a team tags against the same definitions. Spans may not overlap, so every
word carries at most one class.

Exports cover both shapes the downstream tools need: the original **character-span** JSON
and JSONL, and **CoNLL/BIO** token tags, which a BERT token-classification head or
`spacy convert` consumes directly.

See [Manual_Tagger/README.md](Manual_Tagger/README.md).

## Automated taggers

Planned. The intended shape is that each one reads `data/tweets_c.json`, or the CoNLL
export produced from it, and writes predictions back in the same span format so results
can be compared against the hand-labelled gold standard with the same tooling.
