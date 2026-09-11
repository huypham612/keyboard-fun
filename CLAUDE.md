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

* **Speech says `"A... Apple!"`, and the ellipsis is load-bearing.** It does
  two jobs: it buys about 300ms of pause from the engine without needing a
  second utterance, and its sentence break stops a lone letter before a noun
  being read as the word *a*. Plain `"A Apple"` survives on Samantha
  (Enhanced) but is one voice change from saying "uh apple", which is the bug
  this whole area exists to prevent. Do not tidy the ellipsis into a space or
  a comma.
* **Numbers use a full stop and that is correct, not an oversight.** After a
  whole word ("Five.") a full stop already produces the entire pause; after a
  single letter ("A.") the engine reads it as an initial and gives none, which
  is exactly why letters need the dots. Measured: `Five...` vs `Five.` differs
  by 0ms, `A...` vs `A.` by 302ms. So do not "align" numbers to the ellipsis -
  it changes nothing audible - and do not simplify letters down to a full stop,
  which silently removes their pause.
* **The pill shows the word alone**, not the letter too - the letter is
  already on screen at 46vmin beside it.
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
* **Modifier combos pass straight through** when the keyboard is not locked,
  so the adult keeps their shortcuts. Held keys fire once because small
  children lean on keys.
* **Full screen is not optional and there is no checkbox for it.** The
  keyboard lock, the pointer lock and returning home on exit all exist only in
  full screen, so an opt-out would only ever be an opt-out of every protection
  at once. It is requested unconditionally and simply allowed to fail where
  the browser has no full screen (iPhone Safari).
* **`Escape` is never `preventDefault`ed, even while the keyboard is locked.**
  Holding it is the only way out of full screen once the lock is held.
  Swallowing it traps the user. Do not "tidy" this into the blanket
  `preventDefault`.
* **Keyboard Lock is best-effort and must never be relied on.** Firefox has
  no Keyboard Lock API at all, and it only works in full screen even where it
  exists. Swallowing unmapped keys is done unconditionally for every key
  without a modifier, precisely so behaviour does not depend on the lock. A
  regression here shipped once: `'` and `/` reached Firefox and opened Quick
  Find because swallowing was gated on the lock.
* **Every entrance animation is 520ms.** `.pop`, `.pop2`, `.pop3`, `.pop4` are
  picked at random per press for freshness, but a slower one would mean waiting
  to see the picture, because pacing is already gated on the voice. Add
  variants freely; keep the envelope. New ones must also join the
  `prefers-reduced-motion` rule and the `ENTRANCES` list.
* **The note scale is pentatonic, and that is the whole point.** Keys get
  mashed. On a major scale simultaneous presses are dissonant and grate within
  a day;
  on a pentatonic there are no wrong combinations. Do not "complete" `SCALE`
  into a full major scale. Letters, numbers, colours and shapes all draw from
  the same array (with small offsets so the types sound different), so nothing
  can clash with anything else.
* **Test in Firefox, not only Chrome.** They differ on Keyboard Lock, on
  which keys have browser-level meaning, and on speech voices.

## Touch

The on-screen key grid is built only when `(pointer: coarse)` matches, or `?keys`
is in the URL. **A laptop must be completely unaffected** - no grid built, no
layout change, no hint bar hidden.

Pad keys go through the same `ready()` gate as physical keys, or mashing the
grid would bypass the cooldown entirely. They also `stopPropagation()`, or the
tap-anywhere-cycles handler fires a shape on top of every keypress.

Note that `#app` is `overflow: hidden`, so **checking
`documentElement.scrollWidth` will not detect a pad that overflows** - measure
the pad itself. Headless Chrome also clamps `--window-size` to a 500px minimum
width, so a narrow phone viewport has to be faked with a sized iframe or the
screenshot is just a crop of a wider render.

## Theming

Colours live in CSS custom properties on `:root`, with a
`@media (prefers-color-scheme: dark)` block redefining them. **Never give a
colour its only definition inside the dark block.**

The catch: `SKIES` and the colour backdrops are applied as *inline* styles from
JS, which beat the stylesheet, so they cannot be themed by CSS alone. They have
parallel dark values (`SKIES_DARK`, and a 4th entry per colour) chosen by
`isDark()`, and a `matchMedia` listener repaints the current view when the
system flips. If you add anything else that sets a colour from JS, it needs the
same treatment or it will stay light at night.

The colour disc's outline is `--disc-stroke` and must invert with the theme, or
the Black disc vanishes on a dark ground and the White one on a light ground.

`CONFETTI_COLORS` is shared by two things that want opposite treatment at
night. The confetti itself softens to `CONFETTI_NIGHT`, because full-brightness
scatter over the dimmed background undoes the point of dark mode. The **shapes
deliberately keep the vivid palette**: one big solid shape reads better bright
against a dark ground. Don't "fix" the inconsistency by softening both.

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
