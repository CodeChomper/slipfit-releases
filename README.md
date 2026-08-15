# The documentation site

`index.html` is the onboarding guide — one self-contained page and no build step. Open it
directly:

```bash
xdg-open docs/site/index.html          # or just double-click it
python3 -m http.server -d docs/site    # if you would rather serve it
```

**Who it is for.** Someone who has a printer and has never written code to make a model.
It runs from the thesis through installation, the window, the language, fitted cuts, the
three hardware features, the calibration loop and export, and ends with eight worked
examples and the real error messages. Code is glossed line by line rather than assumed —
`def`, the `for` loop and the `name=value` argument each get a plain-English reading at
the point they first appear.

**The window is the documented route; the terminal is a section, not a thread.** Every
task on this page is described as the app does it — menu, panel, button. `workshop.py`
lives in one section under *Going further*, with a table mapping each command back to the
control it replaces. Do not re-scatter commands through the other sections: a reader who
never opens a terminal should be able to finish the page, and the calibration loop in
particular is a sheet with five panels, not a transcript.

**The one thing it fetches** is two webfonts, Archivo Black and Space Grotesk, from Google
Fonts. Everything else — CSS, SVG, the highlighter — is inline. Offline the page falls
back to Impact and the system sans and reads fine; it simply is not the brand. Drop the
two families into the repo and swap the `<link>` for `@font-face` if that trade stops
being worth it.

It is the way *in*. The normative detail stays in the sibling documents — `language.md`,
`modelling.md`, `composition.md`, `printing.md`, `calibration.md`, `shell.md`,
`inspector.md`, `worker.md` — and the site links to each of them by name rather than
restating them.

## The examples are executed, not written down

Every `.slip` on the page lives in [`examples/tour/`](../../examples/tour) and was built
against the kernel before it was quoted. The figures under each one — volume, bounding
size, face count — are what it actually produces. `docs/modelling.md` opens by explaining
why that matters; this page is held to the same standard.

```bash
.venv/bin/python -c "
from slipfit.prelude import model
from pathlib import Path
for f in sorted(Path('examples/tour').glob('*.slip')):
    p = model(f)()
    print(f'{f.name:22} {p.volume}  {len(p.faces())} faces')
"
```

Three of the eight exist to teach a selector that is *wrong* in the obvious form —
`04-finish.slip` and `07-lid.slip` both carry the count that a naive predicate matches
instead. If you change those models, re-run them and update the figures on the page.

`08-composed.slip` includes `examples/tour/parts/standoff.slip`, which is a library and
binds no `part`. The glob above is deliberately non-recursive so it skips `parts/` — a
library has nothing to build on its own, and the loop expects every file it finds to.

## Editing

Everything is inline: tokens at the top of the `<style>` block, then layout, then
components. The `.slip` code blocks are highlighted at load time by the small tokeniser at
the bottom of the file.

**The sidebar and the text travel as one centred block.** `--shell` is
`--side + --measure + --main-pad * 2`, and both `.shell` and `.topbar-inner` carry it with
`margin:0 auto` — so on a wide monitor the nav sits against the text instead of being
stranded at the window edge, and the navbar's contents line up with the column beneath
them. `.topbar` itself stays full-width so its rule still reaches both edges. Changing
`--measure` or `--side` moves all three; do not hard-code a width anywhere else.

**Its vocabulary is generated — do not edit it by hand.** The region between the
`slipfit:vocabulary` markers comes out of the same generator the editor's grammar does:

```
python -m slipfit.lang.grammar --site
```

Run that when the language gains a name, and commit the result. `tests/docs/test_site_vocabulary.py`
fails if the file has fallen behind, which is how this stopped being a list that quietly went
stale — it had already lost four Python keywords before it was generated.

**The page is dark only, and that is a decision rather than an omission.** It is built on
the CodeChomper design system, whose whole visual language is near-black (`#080808`) with
one working accent (`#8cc641`) and orange (`#bf4e30`) for anything that warns. A light
variant would be a different brand, so there is no `prefers-color-scheme` block and no
`data-theme` handling. The two accents also carry meaning here and are not decoration:
green is measured, correct, the thing that fits; orange is warned, nominal, the thing
that bites.

## Pictures

They live in `docs/site/images/`. One is real — `AppDemo.jpg`, the hero shot under *What
this is* — and fourteen places are marked with a `figure.shot` holding a `.shot-ph`
stand-in that names the file, the intended size and what the shot should contain, each
preceded by a `REPLACE ME` comment. To fill one in, drop the image in and replace the whole
`.shot-ph` div with the `<img>` the comment above it already spells out:

```html
<img src="images/calibration-gauges.png" alt="…" width="1400" height="620" />
```

`figure.shot img` is already styled, so nothing else changes and the `figcaption` stays
put. Grep `REPLACE ME` for the list. Screenshots of the app want its own dark UI, which
sits on the same near-black as the page.

**Every `<img>` opens in a lightbox**, wired by tag rather than by class and delegated from
a document click — so a picture dropped in later needs nothing added to it. It is a
`<dialog>`, so the browser supplies the modal, the backdrop, Esc and returning focus; the
enlarged view captions itself from the `figcaption`, falling back to the `alt`. Which is
the argument for writing a real `alt` on each one: it is not only for screen readers here.

Three block styles do three jobs, and mixing them up is the easiest way to make the page
inconsistent:

| Class | Is | Chrome |
|---|---|---|
| `pre.file` | a `.slip` you would save | filename strip, from `data-name` |
| `pre.term` | commands you type | traffic-light dots and a path, from `data-name` |
| `pre.msg` | what the tool said back | 4px top bar — orange with `.bad`, green with `.good` |

## The diagrams

Inline SVG, styled by the `.dia` rules so they follow the tokens — `.eg` for an
edge, `.hd` for one the camera cannot see, `.hi` for the highlighted set, `.fc`
for a shaded face. Nothing is a raster and nothing is fetched.

A diagram in a `figure.solo` is capped at 540px rather than stretched to the full
measure. Its labels are drawn at a fixed size in the viewBox, so a full-width SVG scales
them past the body text they are annotating.

The isometric blocks all use the same projection, from a camera at
front-right-above:

```
screen_x =  0.7071x + 0.7071y
screen_y =  0.3470x - 0.3470y - 0.8670z        (SVG y grows downward)
```

Under it the origin corner is nearest the viewer, the visible faces are `+x`,
`-y` and `+z`, and the three edges meeting the far corner `(0, b, 0)` are the
hidden ones. That is enough to add a panel by hand: project the eight corners,
draw the twelve edges, and give the ones the selector matches `class="eg hi"`.
