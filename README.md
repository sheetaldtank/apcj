# ONOS APC Support Hub — IIT Gandhinagar Library

A four-page static site for the ONOS Article Processing Charge scheme, live at
`library.iitgn.ac.in/apcj/`. No framework, no server-side code, **no external requests of any kind**.

Deploy by uploading `onos-apc-hub-v7.zip` to cPanel File Manager and extracting it.

```
apcj/
├── index.html          Overview, resource cards, five-step process, contacts
├── guidelines.html     Full guidelines, sticky contents sidebar, print stylesheet
├── faq.html            31 questions, live search + topic filters, deep-linkable (#q19)
├── onoslist.html       Journal browser: search, A–Z, facets, sort, paging, CSV export
├── UPDATING.txt        One-page procedure for whoever maintains the list
├── assets/
│   ├── APC_Journal_Titles.csv   ← THE DATA. Replace this to update the list.
│   ├── style.css       Styling source of truth (inlined into the pages)
│   ├── site.js         SITE config, nav, scroll-spy, shared helpers
│   ├── onos-csv.js     Reads the CSV in the browser at page load
│   ├── journals-data.js  Offline fallback snapshot
│   ├── onoslist.js     Journal browser logic
│   └── faq.js          FAQ search/filter logic
└── tools/              Maintenance only — never uploaded to the server
    ├── add-publishers.html   Adds Publisher / OA Type / URL to a fresh ONOS export
    ├── build.py              Inlines style.css into the four pages
    └── csv_to_journals.py    Regenerates the offline fallback
```

---

## Updating the journal list

The whole procedure, no software required:

1. Download the CSV from <https://www.onos.gov.in/apc-support>
2. Open `tools/add-publishers.html` in a browser, load that CSV, click **Start lookup**,
   then **Download completed CSV** — this adds Publisher, OA Type and URL, which the
   official export omits
3. In cPanel → File Manager → `assets/`, upload it as `APC_Journal_Titles.csv`, overwriting
4. Ctrl+F5

The page parses the CSV itself on every load, so the journal count, A–Z strip, all three facets,
the search index and the CSV download rebuild automatically. Counts on the Home, Guidelines and FAQ
pages update at the same moment. **No HTML, CSS or JavaScript needs editing.**

A blue notice at the top of the Journal List confirms the file was read and shows the export date
from inside it. If it turns red, the CSV wasn't found — the site keeps working from the built-in
snapshot and the notice names the cause.

Step 2 is optional. Without it the list still works; the Publisher and Access-type filters simply
stay hidden.

## Current data

426 journals, ONOS export dated 17-08-2026 — 417 with a publisher (31 distinct), 426 with an OA type
(423 Gold, 3 Diamond), 392 with a journal URL, 258 subject tags.

## Editing text and contacts

Everything institution-specific is in one `SITE` object at the top of `assets/site.js` — Nodal
Officer, library email and phone, institute code, endorsement signatory, review date. Change a value
there and it propagates to all four pages via `data-cfg` attributes.

**Still to confirm:** `signingAuth` is set to `'Dean, Academic Affairs'` as a placeholder. The ONOS
admin panel doesn't record who signs the Head-of-Institution endorsement. Confirm with the Dean's
office, change that one value, then delete the three `<span class="tag amber">Confirm locally</span>`
markers in index.html, guidelines.html §5 and faq.html Q19.

## Editing the styling

`assets/style.css` is the source of truth, but the pages don't link to it — the CSS is embedded in
each page's `<style>` block so nothing is fetched at runtime. After any CSS edit:

```bash
python tools/build.py          # re-inline into all four pages
python tools/build.py --check  # exit 1 if any page is stale — for a pre-commit hook
```

Forgetting this is the one easy mistake: the pages keep serving the previous stylesheet.

## Design

Colours and UI conventions were sampled from the Library's own e-Journals Discovery page, so the hub
sits alongside `library.iitgn.ac.in` rather than looking like a separate site: slate-100 ground,
white 12px-radius cards, blue-700 links, blue-600 buttons, pastel subject pills. Brand red `#EC1F28`
and blue `#1971B8` come from the IITGN Library wordmark, which is embedded in the CSS as a 5 KB data
URI so there is no image file to lose during upload. Type is a system font stack — no webfonts.

## Implementation notes

**`onos-csv.js`** handles the official export as-is: UTF-8 BOM, three preamble note lines above the
header, quoted fields, CRLF endings. Two traps it deals with:

- *Comma-bearing subject names.* Subjects are comma-separated, but ~20 Scopus ASJC categories contain
  commas themselves — a plain split turns "Ecology, Evolution, Behavior and Systematics" into three
  bogus tags. Those names are protected in `COMMA_CATEGORIES`; add to that list if a revision
  introduces another.
- *URLs.* Only absolute http/https values are accepted, so a malformed cell can't produce a link that
  resolves against your own domain.

**`onoslist.js`** keeps one `state` object; every control mutates it and calls a single `render()`.
Each record's search text is normalised once at load, not per keystroke. Search is AND-across-tokens
over title, publisher, OA type, both ISSNs and subjects, accent-folded, debounced at 160 ms. Facets
build themselves from the data and hide when a column is absent, so the page adapts to whatever the
CSV supplies. CSV export writes the *filtered* set with only the columns present, UTF-8 with BOM so
Excel opens it correctly. Pressing `/` focuses the search box.

**`faq.js`** caches each question's text at load and filters on `indexOf`. Matches auto-expand while
typing; a topic heading hides when none of its questions survive. `faq.html#q19` opens that answer
directly — useful when replying to email.

**`guidelines.html`** has no JS of its own. Its print stylesheet drops the nav, sidebar and buttons,
so *Print → Save as PDF* produces a clean handout.

## Content accuracy

The process content — eligibility, the eight portal steps, INFLIBNET's invoice address and
GSTIN/PAN, the liability clauses, the 15–20 working day payment window — should be cross-checked
against the official documents before each academic year:

- <https://www.onos.gov.in/public/docs/APC-Guidelines-V1.pdf>
- <https://www.onos.gov.in/public/docs/APC_Manual_v3.pdf>

Update `SITE.reviewed` when you do; it renders in every footer.

Note that OA type is derived from OpenAlex APC data, which is incomplete — a journal shown as Gold
may in fact be Diamond. Publisher names are considerably more reliable.

## Browser support

Current Chrome, Edge, Firefox and Safari. Uses `fetch`, `<dialog>`, `Element.closest`,
`String.normalize`, CSS custom properties and `:has()`. Fully responsive; the facet sidebar stacks
below 940px.
