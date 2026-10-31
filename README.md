# EhabLab — user guide (docs site)

Two self-contained pages. No build step, no dependencies, no CDN.

```
docs/
  index.html          QUICK guide — the one students get (6 blocks, ~400 words/lang)
  manual.html         FULL manual — for whoever maintains the app (20 sections)
  assets/logo.png     hero mark + favicon
  screens/*.png       the 15 app-screen figures used by both pages
  _tools/             how the figures were produced (not used at runtime)
```

`index.html` is the landing page and links to `manual.html` once, at the bottom. The split is
deliberate: a student needs four steps and six screens, not the exam blueprint, the SM-2 internals
or the admin CMS. Keep the quick page short — if a fact is only useful to the person maintaining
the bank, it belongs in `manual.html`.

## Language

The page opens in Arabic (`dir="rtl"`); the pill in the hero switches to English and flips
`lang` / `dir` on `<html>`. The choice is remembered in `localStorage` (`ehablab_docs_lang`).
Every section exists twice, wrapped in `.lang-ar` / `.lang-en`; CSS hides the inactive one, so
there is no runtime templating and the page works with JavaScript disabled (it just stays Arabic).

## Publishing

The guide is live at **https://ehabdoc.github.io/ehablab-docs/**, served from the public repo
`ehabdoc/ehablab-docs`. Pages needs a public repo on a free plan, and `EhabLab` is private, so the
guide is published separately — the question bank never leaves the private repo.

`ehabdoc/ehablab-docs` is a **copy**, not a submodule: this folder is the source of truth. To
republish after editing:

```bash
rm -rf /tmp/ehablab-docs && git clone https://github.com/ehabdoc/ehablab-docs /tmp/ehablab-docs
cp docs/index.html docs/manual.html /tmp/ehablab-docs/
cp -r docs/assets docs/screens /tmp/ehablab-docs/
cd /tmp/ehablab-docs && git add -A && git commit -m "docs: update guide" && git push
```

`_tools/` is deliberately **not** published — it holds `plate.jpg`, a crop of the source PDF's
colour plate section. Only `index.html`, `manual.html`, `assets/` and `screens/` go out.

All links are relative and there is no external asset, so nothing else is required.

## Regenerating the screen figures

`_tools/kit.html` renders the app screens from the real theme tokens (`#0f2a44` navy,
`#0e7c7b` teal, `#0f172a` dark surface) and `_tools/cdp.js` drives headless Chrome over the
DevTools protocol to capture each screen at 2× into `screens/`.

```bash
# 1. serve docs/ so the kit can load plate.jpg
python -m http.server 8898 --bind 127.0.0.1     # from docs/

# 2. start headless Chrome with a debugging port
chrome --headless=new --remote-debugging-port=9222 --window-size=440,960 about:blank

# 3. capture (see _tools/cdp.js for the action format)
node _tools/cdp.js actions.json
```

An action file looks like:

```json
[
  {"type":"size","w":440,"h":960,"dpr":1},
  {"type":"nav","url":"file:///.../docs/_tools/kit.html","wait":2500},
  {"type":"shotel","id":"s-home","path":"screens/home.png","scale":2}
]
```

After capturing, downscale to 720 px wide and quantize, which takes the set from ~1.5 MB to
~860 KB:

```python
from PIL import Image
im = Image.open(f).convert("RGB").resize((720, 1258), Image.LANCZOS)
im.quantize(colors=200, method=Image.MEDIANCUT).save(f, optimize=True)
```

The figures are reconstructions of the app's screens at the current build, not device captures —
the headless browser has no emoji font, so real captures carried tofu boxes. If you update the
app's UI, update `_tools/kit.html` and re-run the two steps above.

## Numbers in the guide

Every count on the page comes from the bundled bank of this build and must be updated together
with the content: 2,402 questions (D1 362, D2 815, D3 382, D4 164, D5 25, D6 236, D7 287,
D8 131), 73 images (60 plates + 13 figures), 468 range hints, 41 unresolved figure gaps,
version 1.0.0+1.
