# ALBEDO

A note-taking app that lives in a single HTML file. No install, no account, no internet required.

Open `albedo.html` in your browser and start typing.

---

## How it works

**Write** — type in the bar at the top, press Enter.

**Search** — start with `?` to filter. Boolean operators work: `?cats AND dogs NOT fish`

**Sort** — type `?~<key>` to filter `key` then sort values alphanumerically. Ascending by default, type `?~d <key>` for descending.

**Tag & color** — put tokens at the start of your note:
```
#journal !blue My thoughts today...
#work ~priority:high Finish the report
```

**Commands** — `&` prefix fetches external content:
- `&dic: word` — French dictionary (Wiktionary)
- `&wiki: subject` — Wikipedia summary
- `&source: topic` — academic papers (Crossref)
- `&meteo: Paris 2024-07-14` — historical weather

**Folders** — type `/projects` to switch folders, click the `/` label to browse.

**Sync** — configure GitHub Gist with `&github: YOUR_TOKEN GIST_ID`, then hit Sync.

---

## Keys

| Key | Action |
|---|---|
| Enter | Save note |
| Shift+Enter | New line |
| Tab | Autocomplete tag / command |
| Escape | Clear / cancel |

---

## Export / Import

- **Json** — exports the current view as a JSON file you can load back
- **Md** — exports as Markdown
- **Load** — imports a JSON file (merge or replace)
- **Clear** — wipes everything (asks first)

---

Your data stays in your browser (IndexedDB). Nothing leaves your machine unless you configure Gist sync.
