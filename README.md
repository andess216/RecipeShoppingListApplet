# Recipe Box

A single-file recipe app with a smart shopping list. No build step, no
dependencies, no framework — one `.html` file with inline CSS and JS. Your
recipes are stored in your browser's `localStorage`.

**Live:** https://andess216.github.io/RecipeShoppingListApplet/

## What it does

**My Recipes** — 20 recipes as expandable cards, each with Ingredients / Steps /
Notes sub-tabs. Add and delete your own. Steps are optional; recipes without them
just don't show that tab.

**Shopping List** — tick the recipes you're cooking this week and get one combined
list. Ingredients merge across recipes (two 400ml coconut milks become one 800ml
line), sort themselves into store categories, and checked-off items sink to a
section at the bottom.

Quantity merging converts within weight (g/kg) and volume (ml/l). Anything it
can't convert is joined with `+` rather than guessing. Ranges like "2–4 inches
ginger" round **up**, so you never under-buy.

**Export / import** — two buttons doing two different jobs:

- **Export list** (Shopping List tab) turns the combined list into plain text and
  opens the iOS share sheet, so you can send it to someone or drop it in Notes.
  On desktop it copies to the clipboard instead.
- **Export backup** / **Import backup** (My Recipes tab) saves every recipe to a
  `recipe-box-backup-YYYY-MM-DD.json` file and loads one back. This is how you
  move your recipes to another device, since nothing syncs on its own.

On import you choose **Merge** (adds only recipes you don't already have, matched
on id then title) or **Replace** (wipes what's there and uses the file). Replace
asks for confirmation first.

## Use it on your phone

1. Open the live URL in Safari on your iPhone
2. Tap **Share** → **Add to Home Screen**

It then launches fullscreen with no Safari chrome, like a native app.

## Where your data lives

Recipes are saved in `localStorage` in whichever browser you opened the app in.
That means:

- **Nothing is uploaded anywhere.** There's no server and no account.
- **Data does not sync between devices.** Your phone and your laptop keep separate
  recipe lists. Whichever copy you add to your home screen becomes the real one —
  use **Export / Import backup** to move recipes between them by hand.
- **Clearing your browser data deletes your recipes.** Export a backup first.

Note that the *published site* is public — anyone with the URL can open the app —
but the recipes you add are only ever on your own device.

## Running it locally

Just open `index.html` in a browser. That's it.

If you'd rather serve it over HTTP:

```bash
python -m http.server 8000
# then visit http://localhost:8000
```

## Files

| File | What it is |
|---|---|
| `index.html` | The app. This is the one that's deployed. |
| `recipe-box.html` | The Claude-artifact variant — see [DEVELOPING.md](DEVELOPING.md) |
| `.nojekyll` | Tells GitHub Pages to skip Jekyll processing |

## Deploying

Pushing to `main` publishes automatically. Pages is configured as
**Settings → Pages → Source: Deploy from a branch → `main` / `root`**.

GitHub Pages on a free account requires the repo to be **public**.

## Editing the code

See [DEVELOPING.md](DEVELOPING.md) — there are a few non-obvious things
(especially the storage-key version bump) that will bite you otherwise.
