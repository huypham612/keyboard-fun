# Keyboard Fun

A keyboard toy for toddlers. One self-contained `index.html`, no dependencies,
no build step, no server. Opening the file is the whole install.

If this checkout contains a `FORK-NOTES.md` or a `CLAUDE.local.md`, read it too:
it carries conventions specific to this copy.

## Hard constraints

* **Stay a single file.** No framework, no `package.json`, no bundler, no CDN
  script tags. It has to keep working from `file://` with the wifi off.
* **Keep it dependency-free.** Plain ES5-style JS, no build step.

## Decisions that look arbitrary but are not

Please don't "simplify" these away without reading why.

* **Speech says "A is for Apple", never "A for Apple".** A lone "A" before
  "for" gets read by speech engines as the word *a*. As the subject of "A
  is..." it cannot be. This was tested against all 26 letters; the wording is
  what removed the need for per-letter pronunciation hacks.
* **Pacing waits for the voice to finish**, with `PRESS_DELAY` only as a floor.
  A fixed delay cannot work: phrases run from 0.33s ("Red!") to 2.0s ("Four.
  Four strawberries!"). `VOICE_TIMEOUT` exists because a browser that never
  fires `onend` would otherwise lock the toy up forever.
* **Keys that do nothing don't spend the cooldown.** Otherwise brushing `tab`
  swallows the real keypress that follows it.
* **`VOICE_NAME` is matched by name, not exact string**, so "Samantha" also
  finds "Samantha (Enhanced)" and prefers it. Exact-matching would silently
  fall back to the browser default on machines without that build.
* **There is deliberately no voice picker.** An earlier one saved a choice to
  `localStorage`, which pinned users to a worse voice build after they
  installed a better one.
* **Modifier combos pass straight through** so the adult keeps their shortcuts,
  and held keys fire once because small children lean on keys.

## Editing content

Everything a parent would change sits in one block at the top of the `<script>`
tag: `LETTERS`, `NUMBERS`, `COLORS`, `SHAPES`, and the tuning knobs. A letter's
picture is normally an emoji but can also be an image (see `pictureItems`).

## Verifying changes

Reading the code is not enough here; render it. Headless Chrome against a
`file://` copy with a stubbed `speechSynthesis` catches what review misses.
Real bugs found this way: a crescent moon that was actually a ring, and a photo
that displayed sideways because of an EXIF orientation tag. Check for JS errors
and for horizontal/vertical overflow at 1440x900 after any layout change.
