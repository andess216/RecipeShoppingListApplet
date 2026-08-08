# Developing

`index.html` is the whole app — inline CSS, inline JS, no build step. Open it in a
browser, edit, refresh.

## Gotchas that bite every time

- **`DEFAULT_RECIPES` (~line 520) seeds first run. The storage key is
  `saved-recipes-v4`.** Bumping that version suffix is how you force existing
  users to pick up changed defaults — otherwise their saved copy wins and your
  edits to `DEFAULT_RECIPES` are invisible. This is the single most common
  "why isn't my change showing up" cause.
- **Saves are debounced 600ms and fire-and-forget** so taps feel instant. Don't
  `await` them in the toggle handlers — that was the original lag bug.
- **`safeSet()` retries failed writes three times with backoff**, then shows a
  banner. Left over from debugging a flaky backend; harmless with `localStorage`,
  but leave it in.
- **`_open` (card expansion state) is stripped before saving.** Don't persist it.

## How ingredient merging works

`parseIngredient()` → `normalizeName()` → `combineQuantities()`.

- Units convert within weight (g/kg) and volume (ml/l). Anything else is joined
  with `+` rather than guessing a conversion.
- Ranges ("2–4 inches ginger") round **up** deliberately, so you don't under-buy.
- Merging is **shopping-list-only by design**. Recipes keep their original
  per-step quantities — while cooking you need "400ml at step 7", not the
  combined weekly total.

## Export / import

Deliberately two separate features that don't share a code path:

- **Shopping list → text** (`buildShoppingText`, `exportShoppingList`). Output is
  for a human to read. It reuses `buildMergedItems()` and `CATEGORY_ORDER`, so the
  text file groups and sorts exactly like the on-screen list — change one and the
  other follows.
- **Recipes → JSON** (`exportRecipes`, `handleImportFile`, `applyImport`). Output
  is for this app to read back on another device.

Notes:

- `exportShoppingList()` tries `navigator.share` (the iOS share sheet), then the
  clipboard, then falls back to downloading a `.txt`. The first two need a secure
  context, so opening `index.html` over `file://` will always take the download
  path. That's expected, not a bug.
- The backup wrapper is `{app, version, exported, recipes}`. `BACKUP_VERSION` is
  currently `1` — bump it if the recipe shape ever changes, and branch on it in
  `handleImportFile`. Import also accepts a bare array, so a hand-written file works.
- **Merge dedupes on id, then on lowercased title.** Since there's no edit
  feature, a repeat title means a re-import rather than a revision. If editing
  ever lands, this rule needs revisiting — it would silently drop edited copies.
- `applyImport('replace')` clears `checkedIngredients` and `openTabs` and prunes
  `selectedForShopping` / `shoppingChecked` down to surviving ids. Skip that and
  you leave tick state pointing at recipes that no longer exist.
- Imported recipes are normalized through `normalizeImported()` — every field is
  coerced to the expected type and a missing `id` is generated. Don't trust the
  file's shape; it may have been hand-edited.

### Testing it

There's no test framework. The logic was verified with a throwaway Node harness
that regex-extracts both `<script>` blocks, stubs `document` / `localStorage` /
`Blob` / `navigator`, and runs the real functions. One gotcha if you rebuild it:
top-level `let`/`const` stay in the vm script's lexical scope and never appear on
the context object, so append a bridge (`globalThis.__app = {get recipes(){...}}`)
to the *same* script string rather than trying to read them from outside.

## The two versions

This app was originally built as a Claude artifact, which provides its own
`window.storage` API. `recipe-box.html` is that variant, and it only runs inside
Claude.

`index.html` is the standalone version. It started as the same file plus three
platform additions:

1. Apple meta tags in `<head>` for "Add to Home Screen" (fullscreen, no Safari
   chrome, theme color)
2. `padding-top: env(safe-area-inset-top)` and `-webkit-text-size-adjust: 100%`
   on `body`, for the iPhone notch
3. A `window.storage` shim near the top of the first `<script>` block that
   implements the same async `get`/`set`/`delete`/`list` interface on top of
   `localStorage`

Because the shim matches the artifact API exactly, all the app logic below it was
originally byte-identical between the two files.

**That is no longer true.** `index.html` has since gained export/import, which
`recipe-box.html` doesn't have. `index.html` is now the only maintained copy;
`recipe-box.html` is kept as a record of where this came from. Don't treat a diff
between them as a list of things to fix.

### Syncing a change made in Claude

If you edit the artifact on claude.ai and want the change live, port the change
across by hand into `index.html`. **Never copy the artifact file over
`index.html`** — you'd drop the storage shim (the app then fails to load any saved
recipes) along with export/import and anything else added since.

To see what you're working with:

```bash
diff recipe-box.html index.html
```

Expect the three additions above *plus* the whole export/import section. If you'd
rather stop maintaining two files, deleting `recipe-box.html` costs nothing.

## Deliberately missing

**Asam Bake** and **Air Fryer Pork Belly** have no method steps. That's not a bug
— the recipes haven't been written down yet. Don't invent them.

## Discussed but not built

A servings scaler, editing existing recipes (currently add/delete only), a "clear
checked items" button, manual non-recipe items on the shopping list, and search.
