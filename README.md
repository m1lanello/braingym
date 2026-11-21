# Daily Brain Gym – build package

A 60-day training app for network engineers: Python, network automation, Anki-style flashcards (ENARSI),
interview practice, Python and Ansible script libraries, quick courses, a lab track, an AI tutor and stats.
Everything lives in **one HTML file** with no server. Progress is stored in your browser (`localStorage`).

## Layout
- `src/parts/` – the app split into ordered pieces (HTML shell, CSS, data files, code sections). Edit these.
- `src/manifest.txt` – the order the parts are joined in.
- `src/pwa/` – icons used for the installable version.
- `build.py` – joins the parts into `dist/Brain_Training_60day.html`.
- `tests/` – Playwright checks (`pip install playwright`, then `playwright install chromium`).

## Build
    python build.py                # dist/Brain_Training_60day.html  (open it, or send it to your phone)
    python build.py --pwa          # also dist/pwa/ for hosting
    python build.py --verify FILE  # confirm the build is identical to FILE

The build runs `node --check` on the JavaScript when node is installed.

## Test
    python tests/test_smoke.py       # every tab opens, no JS errors, no sideways scroll on a phone
    python tests/test_days.py        # all 60 days render, nothing left over from removed subjects
    python tests/test_features.py    # Today page, search, lab/Git lessons
    python tests/test_flashcards.py  # builds a test Anki package, imports it, studies, searches
    python tests/make_fixtures.py OUT_DIR 50000   # a big test deck (65,000 cards) for load testing

## Use it on a phone / offline
A plain `file://` page cannot be installed as an app. Host the contents of `dist/pwa/` over HTTPS
(GitHub Pages, Netlify, Cloudflare Pages – all free), open the address on your phone and choose
"Add to Home screen". After the first visit it works with no connection. The AI tutor is the only part that needs internet.

Progress is stored **per address**: the hosted copy and the local file do not share it.
To move progress, use Stats → Export backup on one and Import on the other.
The app also keeps its own rolling snapshots (last 5) in Stats.

## Not verified here
Playbooks and scripts were checked for syntax and run against stand-ins, not against real
Ansible/Netmiko/napalm/Nornir/TextFSM/pytest installs or real devices. Try them in a lab first.

## Flashcards (v15 additions)
- **Go through**: shows every card of a deck once, in order, no ratings; your place is saved, "Start again" resets it.
- **Ask AI** (study and go-through screens): sends the card to Google Gemini using your own API key (same key as the AI tutor tab). Pick a model or press "Load models" to list what your key supports. Card text/images go to Google; answers can be wrong, check them.
- The 60-day grid is folded away; press **Days** to open it.

## Flashcards
- Import: an Anki package (`.apkg`, exported with "Support older Anki versions" ticked) or a `.txt` / `.csv` / `.tsv` file.
  Images, sound, cloze cards and your Anki review history come with an .apkg. Re-importing skips cards already there.
- Storage: IndexedDB (database `brainGymCards`), not localStorage, so tens of thousands of cards fit. Only two small
  counters (`fcN`, `fcDays`) live in the normal progress data.
- Backup: when flashcards exist, Export writes one `.bgbak` file (JSON lines, gzip, optional AES-256-GCM in 1 MB frames).
  It is written and read in pieces. A wrong passphrase, a damaged file or a cut-off file is rejected before anything changes.
  Without flashcards, Export still writes the plain JSON file, and old backups still restore.
- Card HTML from files is never trusted: it is rebuilt from an allow-list (no scripts, links or external images).
- Not supported: Anki's newest package format (zstd), which browsers cannot read without a large extra library.
