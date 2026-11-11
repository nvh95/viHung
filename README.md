# viHung - vi Boot Camp

An interactive, game-style trainer that teaches beginners to use `vi` on a Linux server. Instead of reading about commands, learners type real keystrokes into a simulated SSH terminal and get instant feedback.

Try it now: <https://vi.hung.dev>


## What's inside

- **Learn, then practice.** Each lesson opens as a short slide deck: one idea or one key per slide, advanced with Next, → / Enter, a tap or a swipe. Key slides let you try the key on your keyboard, and a quick check must be answered correctly before moving on. Then a separate practice screen holds only the terminal and drills. Lessons you have already seen open straight to practice; "Review the concepts" goes back to the slides.
- **12 lessons, 46 drills**, in order:
  1. Get out alive (`:q`, `:q!`, `Esc`)
  2. Normal mode and Insert mode
  3. Saving and quitting (`:w`, `:wq`, `:x`, `ZZ`)
  4. Moving with `h j k l` and counts
  5. Jumping: `w b e`, `0 ^ $`, `gg G`, `:42`, `:set nu`
  6. Deleting: `x`, `dw`, `dd`, `D`
  7. Undo and redo (`u`, `Ctrl-R`)
  8. More ways into Insert mode (`a A I o O`)
  9. Copy and paste (`yy`, `p`, `P`)
  10. Search (`/`, `?`, `n`, `N`)
  11. Replace (`r`, `cw`, `:%s/old/new/g`)
  12. Server missions: `sudo vi`, swap files (E325), frozen terminal, suspended vi, editing `sshd_config`
- **Scoring**: each drill has a par (the minimum number of keys). Finishing at or under par earns 3 stars; stars raise your rank.
- **Goal panel**: shows the target text with lines that still differ highlighted, plus a checklist (text matches, saved, vi closed).
- **Mistake coach**: explains what went wrong when a learner types in Normal mode, types `:wq` inside Insert mode, hits E37, edits a root file without sudo, presses `Ctrl-S`, `Ctrl-Z` or `Ctrl-C`, leaves Caps Lock on, or meets a leftover swap file.
- **Arcade**: two 60-second games, *Cursor Rush* (reach targets fast) and *Line Surgeon* (delete junk lines, keep good ones).
- **Key quiz**: type the exact keys for 10 random tasks.
- **Key card**: every command from the lessons on one page.

## Running it

It is a single self-contained HTML file with no build step and no dependencies besides Google Fonts.

```bash
open index.html
```

Or serve the folder and open it in a browser:

```bash
python3 -m http.server 8000
```

Then visit http://localhost:8000. Click the terminal to focus it and type on your keyboard. On phones, buttons below the terminal provide Esc, Enter, Backspace and the Ctrl keys, but a physical keyboard works best.

Progress (stars, arcade best scores, current lesson) is stored in the browser's `localStorage` under the key `viBootCamp.v1`. Clearing site data resets it.

## SEO and social sharing

The site is deployed to https://vi.hung.dev/ by CI on every push to `main`. If the domain changes, replace that URL in `index.html` (canonical link, Open Graph, Twitter and JSON-LD tags), `sitemap.xml` and `robots.txt`. Social networks need absolute image URLs.

- `index.html` has a title and description, a canonical link, Open Graph and Twitter Card tags, `LearningResource` structured data (JSON-LD), and static lesson content inside `<main>` that search engines can read before the app starts.
- `og-image.png` (1200×630) is the preview image shown when the link is shared on social media. Its source is `og/og-image.html`. To re-render it after editing:

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new --hide-scrollbars --force-device-scale-factor=1 --window-size=1200,630 --virtual-time-budget=4000 --screenshot="$PWD/og-image.png" "file://$PWD/og/og-image.html"
```

- `favicon.svg` is the source icon. `favicon.ico` (16, 32, 48 and 256 px) and `apple-touch-icon.png` (180 px) are rendered from it with `rsvg-convert` (`brew install librsvg`).
- `sitemap.xml` lists the page and `robots.txt` points crawlers to it. Submit it in Google Search Console.

Social sites cache previews. After changing the image, refresh it with the [Facebook Sharing Debugger](https://developers.facebook.com/tools/debug/) or the [LinkedIn Post Inspector](https://www.linkedin.com/post-inspector/).

## How it works

Everything lives in `index.html`:

| Part | What it does |
| --- | --- |
| `Vi` class | A small vi engine: Normal, Insert, command-line and search modes; motions, operators (`d`, `c`, `y`) with counts, registers, undo/redo, `:w`/`:q`/`:wq`/`:x`, `:set nu`, `:s` and `:%s`, `/` and `?` search. |
| `Term` class | A fake shell around vi with a virtual file system: `vi`, `sudo vi`, `cat`, `ls`, `fg`, root-owned files, swap-file prompts, `Ctrl-S` freeze with queued keys, and `Ctrl-Z` suspend. |
| `Player` class | Renders a terminal, captures keys, tracks targets and keystrokes, checks the drill's goal and shows the win screen. |
| `LESSONS` array | Lesson text, key tables, common-mistake notes, quick checks and drills. Each `intro` paragraph and each `keys` row becomes its own slide (`buildSlides`). |
| Views | Lessons, Arcade, Key quiz and Key card, switched by the URL hash (`#lessons`, `#arcade`, `#quiz`, `#keys`). |

### Adding a drill

Drills are plain objects inside a lesson's `drills` array:

```js
{
  task: 'Change <b>yes</b> to <b>no</b>. The cursor is on the y.', // instruction (HTML)
  file: 'sshd_config',            // file name shown in vi
  content: 'PermitRootLogin yes', // starting text
  cy: 0, cx: 16,                  // starting cursor (optional)
  want: 'PermitRootLogin no',     // expected text
  par: 5,                         // minimum keys for 3 stars
  hint: 'cw no Esc.',             // shown by "Show hint"
  tip: 'Optional note on the win screen.',
}
```

Other options:

- `saveQuit: true`: the file must be saved with `want` and vi closed.
- `targets: [[y, x], ...]`: cursor-movement drill; land on each square in order.
- `start`: `'vi'` (default), `'insert'`, `'shell'`, `'frozen'` or `'stopped'`.
- `files: { path: { content, root, swap } }` with `path`: multi-file or shell drills; `root: true` needs sudo to save, `swap: true` triggers the E325 prompt.
- `sudo: true`, `nu: true`: open vi as root or with line numbers on.
- `pre: v => { ... }`: change the buffer after opening (for example, leave unsaved edits or undo history).
- `check: t => boolean` and `chips: [[label, t => boolean], ...]`: a custom goal and checklist.

After changing drills, solve each one yourself in the browser and make sure the par is reachable.

## Limitations

The simulator covers what beginners need and leaves out a lot of vim: no visual mode, no `.` repeat, no text objects (`ciw`), no macros, no multiple registers, and no real regex differences between vim and JavaScript in search and substitute.
