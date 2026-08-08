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

## The two versions

This app was originally built as a Claude artifact, which provides its own
`window.storage` API. `recipe-box.html` is that variant, and it only runs inside
Claude.

`index.html` is the standalone version. It's identical except for three additions:

1. Apple meta tags in `<head>` for "Add to Home Screen" (fullscreen, no Safari
   chrome, theme color)
2. `padding-top: env(safe-area-inset-top)` and `-webkit-text-size-adjust: 100%`
   on `body`, for the iPhone notch
3. A `window.storage` shim near the top of the `<script>` block that implements
   the same async `get`/`set`/`delete`/`list` interface on top of `localStorage`

Because the shim matches the artifact API exactly, **all the app logic below it is
byte-identical between the two files.**

### Syncing a change made in Claude

If you edit the artifact on claude.ai and want the change live:

```bash
# overwrite recipe-box.html with the new artifact export, then:
diff recipe-box.html index.html
```

The diff should show *only* the three additions listed above. Port your actual
change across by hand, or re-apply those three additions to the new file and save
it as `index.html`. Don't just copy the artifact over `index.html` — you'll drop
the shim and the app will fail to load any saved recipes.

## Deliberately missing

**Asam Bake** and **Air Fryer Pork Belly** have no method steps. That's not a bug
— the recipes haven't been written down yet. Don't invent them.

## Discussed but not built

A servings scaler, editing existing recipes (currently add/delete only), a "clear
checked items" button, manual non-recipe items on the shopping list, search, and
text export.
