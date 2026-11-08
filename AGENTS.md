# Plant points — app guide

These instructions apply to the Plant points app (`index.html`) and its food icon assets. They do not govern unrelated files in Downloads.

## Purpose and behaviour

Plant points is a mobile-friendly, single-page tracker of the variety of plant foods eaten each week. It tracks whether a food was eaten, not portions, calories, meals or daily intake.

- The weekly target is 30 plant points (`TARGET`). Weeks begin on Monday using local dates.
- Tapping a food toggles it for the selected week. A food counts once per week.
- Fruit, vegetables and herbs, legumes, and nuts and seeds earn 1 point per food. Items in the separate tea, coffee and spices group earn ¼ point.
- The score ring, achievement levels, badges and streak encourage reaching the target. The highest named level starts at 40 points.
- Users can browse past weeks, choose a date, and return to the current week. Future weeks are disabled.
- Clearing a week and removing a custom food require a second tap within three seconds.

## Views

### This week

Shows the score, seasonal suggestions, food search and category grids. Search matches the built-in catalogue and an additional catalogue, recognises aliases and simple singular/plural variants, and can add new foods to a selected category. Adding a custom food also marks it eaten for the selected week.

Seasonal suggestions use a hard-coded Central European/German harvest calendar. They reflect the current calendar month, including while viewing a historical week.

### Nutrients

Shows a qualitative estimate based on which selected foods have entries in `FN`. Each mapped food contributes once to each of its nutrient codes: zero sources is “Missing,” one is “Some,” and two or more is “Covered.” Recommendations prioritise foods that fill gaps.

This is not a measurement of nutrient intake or a diagnosis. Keep the existing explanatory text about limitations. Foods absent from `FN`, including custom foods, do not contribute. Pak choi, navy beans and parsley root currently have no `FN` entries despite counting towards plant points. Tea, coffee and spices map only to the app’s antioxidant code.

### Trends

Shows the current week and preceding 11 weeks, comparisons with the selected week’s predecessor, best week, average over nonzero weeks in that 12-week window, target achievement count, and the current streak. Chart bars open the corresponding week.

## Files and execution

- `index.html`: the complete app, including HTML, CSS, JavaScript and embedded image data. No build step, package manager or framework is required.
- `icons/*.png`: seven resized source images, each fitting within 96 × 96 pixels, with transparency and proportions preserved. Together they are about 82 KB.
- `icons/originals/*.png`: full-resolution originals retained for future iteration. Preserve these files.
- `index-before-food-icons.html`: backup from before custom icons were embedded. It is not the current app.

Open `index.html` in a modern browser to use the local app. A stable local web origin can also be used during development. Browser storage can differ between file URLs, server origins, browsers and profiles; use the app’s backup export before changing how it is opened.

The page requests Bricolage Grotesque from Google Fonts and has system font fallbacks. App logic and food icons are contained in the file. Mobile home-screen metadata is present, but there is no service worker or offline cache implementation.

## Code map

| Symbol | Responsibility |
| --- | --- |
| `GROUPS` | Built-in foods, group titles, point values, default and per-food emoji |
| `PTS`, `EMO`, `rebuildMaps()` | Derived point and emoji lookup tables, including custom foods |
| `EXTRA_SRC`, `EXTRA` | Additional foods available through search |
| `ALIAS`, `canon()` | Canonical food names and search spelling variants |
| `cap()` | Display labels; specifically displays `bok choy` as “Pak choi” |
| `SEASON`, `WINTER` | Month-based seasonal food lists |
| `NUTN`, `NGROUPS`, `FN`, `NHINT` | Nutrient labels, groups, food mappings and explanatory hints |
| `FOOD_ICONS`, `renderFoodIcon()` | Embedded image icons with emoji fallback |
| `toggle()`, `addCustom()`, `buildTiles()` | Food selection, custom catalogue and tile construction |
| `render()` | Updates selected-week state, score, seasonal foods, trends and nutrients |
| `persistLocal()`, `save()` | Browser storage and scheduling optional cloud saves |
| `initSync()`, `mergeRemote()`, `flush()` | Optional host-provided cloud persistence |

## Data and persistence

Food keys are lowercase names; multiword names contain spaces. They are persistent identifiers, so do not casually rename them. Pak choi deliberately retains the `bok choy` key; `pak choi` and `pakchoi` are search aliases. Navy beans uses `navy beans`; parsley root uses `parsley root`.

The in-memory structures are:

- `data`: Monday date strings (`YYYY-MM-DD`) mapped to arrays of selected food keys.
- `custom`: group titles mapped to arrays of `{n: name, e: emoji}`.
- `wts`: per-week change timestamps, including timestamps for cleared weeks.
- `cts`: timestamp for the custom catalogue as a whole.

Browser `localStorage` keys are `plantpoints.v2` (weekly data), `plantpoints.custom` (custom foods), and `plantpoints.meta` (`wts` and `cts`). The loader also supports migration from `plantpoints.v1`. Keep these keys and compatibility paths unless a deliberate migration is implemented.

Backup export places JSON shaped as `{v:1, data, custom}` into the backup text area. Restore validates selected fields and replaces the current data and custom catalogue; it is not an additive import. Backup files do not include icons or sync timestamps.

Cloud sync is optional and depends on the embedding host providing `window.claude.use`. The app requests `user` and `db`, resolves the user ID, and uses `data/users/<uid>/state`. A standalone browser ordinarily uses local storage only. Do not assume that opening this HTML enables account sync.

Cloud payloads are `{v:3, data, wts, custom, cts, updated}`. Initial remote data is merged using newer per-week timestamps and a newer catalogue timestamp, then the resulting state is written. Saves are debounced by 600 ms; another change during a save schedules a follow-up. The app attempts a pending flush when hidden. There is no continuous remote subscription. Preserve timestamp handling so cleared weeks are not resurrected.

## Food icons

Custom transparent PNG icons exist for plum, red cabbage, beetroot, radish, peas, edamame and tofu. `FOOD_ICONS` maps their canonical keys to base64 PNG data URLs embedded in the HTML. The app does not load images from the `icons` directory at runtime.

Always use `renderFoodIcon()` for food imagery. It creates an image for these seven foods and otherwise uses the emoji from `EMO` or the provided fallback. It is used in grids, search, seasonal suggestions, nutrient recommendations and nutrient source lists.

CSS sizes `.food-icon` to 1 em, matching the surrounding context. Images next to visible food labels are decorative; source-list images receive food-name alternative text. Preserve this distinction.

When revising icons, retain the high-resolution originals, derive small transparent assets, and update the corresponding embedded data URLs. Replacing a PNG on disk alone does not update the app. Preserve emoji fallbacks in `GROUPS`.

## Editing guidance

- Keep the app a single HTML file with embedded icons unless the user requests a structural change.
- Preserve existing saved data and canonical food keys. Adding built-in foods should not silently discard custom foods or weekly history.
- For a new main-list food, update `GROUPS`; add aliases where useful. Remove duplicate search-only entries when promoting a food into the main list.
- Nutrient mappings and seasonal membership are separate from plant-point scoring. Add them only when supported; do not invent nutrition claims to complete a mapping.
- Insert user-provided names with `textContent`, not HTML interpolation. Keep custom text out of executable markup.
- Preserve responsive sizing, light/dark colours, reduced-motion support, keyboard behaviour and accessible control states.
- Save a backup before replacing the working Downloads HTML. Do not overwrite the original icon assets as an incidental cleanup.

## Verification

There is no bundled automated test suite or build command. Scale verification to the change:

1. Check JavaScript syntax after script edits, avoiding unnecessary output of the large embedded image map.
2. Open the app and check any affected views at narrow and wide widths.
3. For food changes, verify search aliases, correct category, point value, toggling and persistence after reload.
4. For icon changes, verify all image-using views, transparent backgrounds, sensible dimensions and emoji fallback for other foods.
5. For persistence changes, use isolated test storage and check export/restore, historical weeks and cleared-week timestamps. Do not reset the user’s actual browser data for testing.
6. Test host cloud integration only in an environment that supplies the expected APIs; standalone local operation does not validate sync.
