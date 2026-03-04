# TODO (SANS ORDRE PARTICULIER)

CONTINUOUS IMPROVEMENTS :

* Refactoring du code de base pour résilience et optimisation des performances
* Refactoring du code pour simplicité, lisibilité et maintenabilité
* Revue de sécurité

---

IDENTIFIED ISSUES (from code review) :

Critical :
* `script-src 'unsafe-inline'` in CSP — known architectural trade-off (requires server-side nonce to fix), document explicitly
* `sys.init()` partial failure — if IndexedDB is corrupt, app continues with empty state and risks overwriting data. Should freeze UI and prompt user.
* `sync()` — no AbortController / timeout on fetch() calls. A stalled GitHub request silently hangs the sync operation.

Moderate :
* `edit(id)` — no re-entry guard: calling on an already-editing block inserts a second <textarea>. Fix: check if element is already a textarea before replacing.
* `wipe()` / `loadCartridge()` — use native `confirm()` / `alert()` dialogs (unstyled, blocks UI thread, suppressed on mobile). Replace with custom modal.

Minor :
* `Math.random()` in ID generation — risk of collision if two blocks are created in the same millisecond. Replace with `crypto.randomUUID()`.
* No automated tests — `sys.parse()`, `sys.format()`, and the boolean query engine have no test coverage.
* Single-file monolith growing — at ~1850 lines, refactors are increasingly risky without test coverage.
