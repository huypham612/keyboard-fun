# Keyboard Fun

A keyboard toy for toddlers, for the age where mashing a laptop keyboard is
the entertainment. Every key press puts a big letter, a big picture and a
friendly voice on the screen.

**[Play it here](https://huypham612.github.io/keyboard-fun/)** — nothing to
install, it runs in the browser.

## How to play

Open the link above, or download `index.html` and double-click it. Either way
that's the whole install: one file, no dependencies, no build step and no
server. The downloaded copy works with the wifi off.

Click **Let's play!** once. That click is what lets the browser use its voice,
so it can't be skipped.

| Key | What happens |
| --- | --- |
| `A`–`Z` | The letter, a picture, and "A for Apple!" |
| `0`–`9` | The digit, that many objects (3 shows three balloons), and "Three. Three balloons!" |
| `↑` or `→` | The next color |
| `↓` or `←` | The color before |
| Space bar | The next shape |
| Click / tap | Alternates: a shape, then a color, then a shape... |
| Everything else | Nothing, on purpose. |

Eleven colors: red, orange, yellow, green, blue, purple, pink, brown, gray,
black, white. They wrap around, so a child can hold one direction and go round
and round.

Fourteen shapes: circle, square, triangle, rectangle, star, heart, oval,
diamond, moon, pentagon, hexagon, octagon, arrow, cross.

Press `esc` to go back to the start screen. Leaving full screen always returns
there, however it happened, so the toy is never left running in a window where
the keyboard and pointer locks do not apply. Getting back in means clicking
**Let's play!** again.

## The notes

Every press also plays a soft note, so mashing the keyboard makes a tune as
well as an alphabet. Letters walk up the scale, and numbers, colours and shapes
each start from a different point so they don't sound alike.

The scale is **pentatonic**, which means no two notes can clash however many
she hits at once. Knobs at the top of the `<script>` tag:

```js
var MUSIC       = true;  // set false for speech only
var NOTE_VOLUME = 0.20;  // it has its own slot, so it can be present
var NOTE_DECAY  = 0.22;  // seconds; short enough to finish before the word
var VOICE_GAP   = 140;   // ms of silence between the note ending and the word
```

The note and the word take turns rather than overlapping: the voice waits for
the note to finish ringing, then leaves `VOICE_GAP` of real silence, so a press
reads as chime, beat, word. Shortening the note automatically brings the voice
in sooner - only the silence is set by hand. With `MUSIC` off there is no note
to wait for and the voice starts immediately.

Don't push `VOICE_GAP` far past 200ms. Beyond that the chime and the word stop
sounding like one event and just feel slow, and because nothing else can happen
until the sentence ends, every extra millisecond is another millisecond of her
presses being ignored.

If it still feels busy, lower `NOTE_VOLUME` first, then `NOTE_DECAY`. The notes are plain sine waves and stop at G5 on purpose: the
octave above that is where a note starts cutting through speech rather than
sitting under it.

The **Sound** checkbox on the start screen governs the notes and the voice
together, so unchecking it gives full silence.

## Light and dark

The theme follows the system setting automatically, and switches live if macOS
flips at sunset - no reload, no setting in the app. Evening play is dim rather
than a wall of white light.

Dark backgrounds are muted rather than black, since a white letter on pure
black is harsher on the eye than on a soft dark ground. Each letter keeps its
own background hue across the switch, so the colour cue it is learned by
survives. The colour discs gain a light outline in dark mode, which is what
keeps Black and White visible.

## Notes for the grown-up

* Holding a key down fires once, not fifty times, and presses that arrive
  inside the `PRESS_DELAY` window are dropped rather than queued, so the toy
  never runs on for a minute working through a backlog.
* `Cmd` / `Ctrl` / `Alt` combos are passed straight through, so your own
  shortcuts still work.
* Each letter always gets the same background colors, giving a second cue to
  recognize it by.
* **Every key without a modifier is swallowed** in all browsers, so unmapped
  keys do nothing rather than reaching the browser. That covers `'` and `/`
  (Quick Find in Firefox), `F5`, `Tab` and the rest. Press `esc` to leave full
  screen.
* **In Chrome or Edge, full screen additionally takes a keyboard lock**, which
  catches modifier shortcuts too (`Cmd+W`, `Cmd+R`). **Firefox has no such API**,
  so there `Cmd+W` still closes the tab. If she keeps finding those, play in
  Chrome:

  ```
  open -a "Google Chrome" index.html
  ```
* In full screen the app takes a **pointer lock**: the cursor is removed from
  the system entirely, so moving the mouse or trackpad goes nowhere at all - it
  cannot reach the menu bar, the Dock, a hot corner or a second display.
  Clicking still works. Leaving full screen gives the pointer straight back.
  If the browser refuses the lock the toy still works, just with a visible
  cursor.
* **Trackpad gestures the browser owns are blocked**: two-finger swipe (which
  is back/forward, and on a `file://` page "back" leaves the toy), pinch to
  zoom, scrolling and double-click zoom.
* **Gestures macOS owns are not**, because the trackpad driver consumes them
  before the browser sees anything. Turn these off in
  *System Settings > Trackpad > More Gestures* if she finds them: swipe between
  pages, swipe between full-screen apps, Mission Control, App Expose, Launchpad,
  Show Desktop.
* macOS keeps some keys for itself that **no web page can intercept**. If she
  finds them, turn them off in System Settings:

  | What she hits | Turn it off in |
  | --- | --- |
  | Spotlight (`Cmd+Space`) | Keyboard > Keyboard Shortcuts > Spotlight |
  | Mission Control / Launchpad (`F3`, `F4`) | Keyboard > Keyboard Shortcuts > Mission Control |
  | Brightness and volume keys | Keyboard > "Use F1, F2, etc. as standard function keys" |
  | Trackpad corner triggering things | Desktop & Dock > Hot Corners (set all to `-`) |

  `Cmd+Q` and `Cmd+Tab` cannot be disabled by any of this.
* Full screen hides the tabs and the address bar, but it cannot stop `Cmd+Q`.
  If you want it properly locked down, launch it in a kiosk window:

  ```
  open -na "Google Chrome" --args --kiosk --app="file://$PWD/index.html"
  ```

## Choosing the voice

The app asks for one voice by name:

```js
var VOICE_NAME = "Samantha";
```

It matches on the name, so "Samantha" also finds "Samantha (Enhanced)" and takes
that better build when it is installed. If no Samantha exists on the machine at
all, the browser falls back to its own default rather than picking something
strange. To use a different voice, change that one string.

Changing the macOS **System voice** in Settings does *not* affect this app. The
browser keeps its own voice list.

### Which one

Among what is installed on this Mac, **Samantha** is the best US English voice,
which is why Samantha is the one named. Karen (Australian), Tessa (South African),
Moira (Irish) and Daniel (British) are comparable quality with an accent, if you
want to try one in `VOICE_NAME`. Avoid the Novelty voices (Zarvox, Bahh,
Bubbles and friends): a toddler might enjoy them, but they will not teach what
a letter sounds like.

**The Siri voices in the System Settings menu cannot be used.** They are
reserved for macOS itself and are not handed to browsers or to any other app.
Of the 184 voices installed here, none are Siri, and asking for one directly
fails outright (`Speaking failed: -241`). So ignore that submenu.

### The real upgrade

Every voice on this Mac is currently the **compact** build, which is the rough,
slightly robotic one. macOS has much better builds of the same voices as a free
download. On macOS 26 the button is called **Customize...**, not "Manage
Voices..." as it was on older versions:

1. System Settings -> Accessibility -> **Read & Speak**
2. Click the **System voice** popup
3. Scroll the menu all the way past the Novelty group (Albert ... Zarvox). The
   list is taller than the screen, so **Customize...** sits below Zarvox, out
   of sight until you scroll
4. In the sheet, find Samantha under English (United States) and tick the
   **Enhanced** or **Premium** entry. It shows you the download size, and a
   **Play** button to hear a voice before committing

Once downloaded, the app prefers the better build automatically, no code change
needed. This is by far the biggest improvement available, much bigger than
switching between the compact voices.

## Pacing and pronunciation

The knobs sit at the top of the `<script>` tag:

```js
var PRESS_DELAY    = 400;    // shortest gap between presses, even for a short word
var WAIT_FOR_VOICE = true;   // and also wait until the whole phrase has been said
var VOICE_TIMEOUT  = 4000;   // safety net if a browser never reports the end
var VOICE_RATE     = 0.9;    // 1 is normal speed
var VOICE_PITCH    = 1.1;    // 1 is normal, higher is brighter
```

A single fixed delay could not work here, because the phrases are wildly
different lengths: "Red!" takes 0.33s to say, while "Four. Four strawberries!"
takes 2.0s. A gap long enough for the numbers makes the colors feel dead, and a
gap short enough for the colors cuts the numbers off halfway.

So the toy waits for the **voice itself** to finish, with `PRESS_DELAY` as a
floor underneath. Everything shares it: letters, numbers, shapes, colors and
clicks. Set `WAIT_FOR_VOICE` to `false` if you would rather have the fixed gap
only, and raise `PRESS_DELAY` to slow things down further.

Two details worth knowing:

* Keys that do nothing (tab, enter, punctuation) don't spend the timer, so
  brushing one won't swallow the real key that follows it.
* If a browser ever loses an utterance and never reports its end, the toy would
  lock up. `VOICE_TIMEOUT` releases it after 4 seconds regardless.

### Why "A... Apple!"

The voice says the letter, pauses, then the word, and the pill shows only the
word since the letter is already on screen beside it.

The ellipsis is doing real work. It buys around 300ms of pause out of the
speech engine without needing a second utterance, and its sentence break is
what stops a lone letter before a noun being read as the word *a* - the bug
that made "A" sound wrong in the first place. Plain "A Apple" happens to
survive on this voice, but a different voice would say "uh apple" again, so
keep the dots.

## Changing the words

Everything editable sits in one block near the top of the `<script>` tag in
`index.html`, marked `CONTENT`. Each entry is `KEY: ["Word", "emoji"]`:

```js
A: ["Apple", "🍎"],
```

Swap in your child's own favorites (their name, the family pet, a food they
like) and reload the page. Numbers take a third value, the plural noun the voice says:
`"3": ["Three", "🎈", "balloons"]`.

`X` is the awkward one. It currently says "X for X-ray" with 🩻, because there
is no xylophone emoji. Change it if you'd rather.
