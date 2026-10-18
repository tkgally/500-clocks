# Five Hundred Clocks

A gallery of 500 animated clocks, each showing the current time in a different way: number systems, languages, historical instruments, machines, games, crafts, natural phenomena, cellular automata and mathematical constructions. Hovering over a clock shows its name and a one-line description; clicking it opens a larger view with notes on how to read it and how it works.

The whole gallery is one self-contained HTML file. It uses no external libraries, fonts, images or network requests, and it runs when opened directly from disk.

**Authorship.** All of the code and all of the text in the gallery and in the build kit were written by Claude Opus 5.5 (Anthropic’s AI model), working under the direction and supervision of Tom Gally. Tom wrote the prompts, reviewed the results and decided what to keep; he did not write or edit the code. The explanations in each clock’s “How to read it” and “How it works” panels were also written by Claude. They were researched and checked during the build, but they may still contain mistakes, and corrections are welcome.

## Using the page

Open `index.html` (the same file as `five-hundred-clocks.html`) in a current desktop or mobile browser.

- The clocks appear in a different random order on each load.
- The toolbar at the top has a grid-size slider, a category menu, a search box (press `/` to jump to it), pause (also the space bar), a speed control (1× to 3600×), a slider that shifts the time up to twelve hours either way, and a “Now” button that returns to the present.
- In the large view, the arrow keys move to the previous or next clock (within the current filter), and Escape closes it.
- Clocks based on the Sun or Moon use an approximate location estimated from the browser’s time zone; the page never asks for your location.

## How the page is structured

The HTML file has five parts, in this order:

| Part                     | What it contains                                                                                                       |
|--------------------------|------------------------------------------------------------------------------------------------------------------------|
| `head`                   | Page styles and the toolbar markup                                                                                     |
| `core.js`                | The clock registry (`defineClock`), the categories, the shared time object, and a helper library `U`                   |
| `part1.js` … `part30.js` | The clocks: each part file defines about 20 clocks                                                                     |
| `app.js`                 | The gallery itself: time control, random hanging order, filter and search, rendering scheduler, tooltip and large view |
| `tail`                   | Closing tags                                                                                                           |

Each clock is a plain JavaScript object:

```js
defineClock({
  id: 'nixie', title: 'Nixie Tubes', category: 'mechanical',
  blurb: 'One sentence for the tooltip.',
  read:  'How to read the time (HTML).',
  about: 'History, mechanism or mathematics (HTML).',
  fps: 30,                              // optional cap on redraws per second
  init(W, H, T) { return state; },      // builds cached artwork
  draw(g, W, H, T, S) { /* paints the whole square */ },
});
```

`T` is the time object (hours, minutes, seconds, fractions, date, and the elapsed time since the last frame). Because the page can be paused, scrubbed and run at up to 3600×, every clock derives what it shows from `T` rather than from its own counter, and simulations re-seed themselves when the time jumps.

The helper library provides small utilities that many clocks share: seeded random numbers, easing curves, bitmap and seven-segment digit drawing, font stacks, and simple astronomy (the Sun’s position, sunrise and sunset, the Moon’s phase, sidereal time and the equation of time).

`app.js` draws only the cards that are on or near the screen, gives the gallery a per-frame time budget, and drops to 30 redraws per second on slower machines. Cards far from the screen release their canvases and cached artwork, so memory use stays at about 40 MB of canvases however many clocks there are.

## How the images are made

There are no image files and no images from an image-generating model. Every clock is drawn by its own JavaScript on an HTML `<canvas>` using the browser’s 2D drawing API: paths, gradients, text and pixel operations.

- In `init`, a clock paints its static parts (a dial, a wooden case, a paper texture, a background scene) once into off-screen canvases. Textures such as wood grain, paper fibre, stone or fabric are generated from seeded random noise, so they look the same every time.
- In `draw`, called up to 60 times a second, it copies those cached layers and paints only the parts that move: hands, digits, particles, simulations.
- Text uses the fonts already installed on the viewer’s computer. Scripts that may be missing on some systems (runes, sitelen pona, Moon type, planet symbols and others) are drawn as vector shapes instead of characters.
- Simulations (cellular automata, fluid and particle models, ray tracing and so on) are computed in the browser at low resolution and scaled up, and they are arranged so that the time can always be read.

Because the artwork is code, the same clock renders sharply at the 140-pixel gallery size and at full screen.

## How it was built

The gallery was built over four working sessions with Claude in October 2026, after an earlier separate experiment in September. In each session Tom gave a prompt and Claude did the design, coding, testing and documentation, mostly by acting as an orchestrator: it wrote a specification and assignments, launched several Claude subagents in parallel to write about 20 clocks each, reviewed their output, sent weak clocks back for revision, and assembled the final file.

Testing used a headless Chromium browser (Playwright). For every clock, a test script ran time jumps, midnight and noon crossings, 3600× fast-forward and pause at two sizes, recorded errors and drawing cost, and saved snapshots at fixed times. The orchestrating Claude looked at contact sheets of these snapshots to check that each clock was legible and showed the right time, and read the language clocks at awkward times such as midnight and quarter to the hour. A whole-page test checked the count, duplicate ids, the random order, memory use, the filter, every large view and the browser console.

### The prompts

Summaries of Tom’s prompts, lightly cleaned up. Short follow-up messages asking about progress are mentioned only briefly.

**September (earlier experiment, separate code).** “Create an interactive static website that can be downloaded as a ZIP file and has no external dependencies for the following idea: ‘A hundred clocks. A gallery of unusual and creative ways to display the current time, such as binary, the Edo-period variable-hour system, word clocks in various languages, and cellular automata.’” Claude produced a multi-file site of 100 clocks without subagents. The present gallery does not use its code, but the idea carried over.

**Session 1 (15 October): the first hundred.** A detailed specification for “A Hundred Clocks: A living gallery of unusual, creative, and beautiful ways to display the current time”, with zero external dependencies:

- exactly 100 distinct clocks updating in real time, spread across six kinds (cellular automata and algorithms; historical and cultural horology; word and typographic clocks; physical and mechanical analogies; cosmic and biological clocks; generative and mathematical clocks), with many examples named for each;
- no titles, captions or numbers on the gallery cards other than those that are part of a clock;
- a tooltip with the title and a one-sentence description on hover;
- on click, a large view split side by side: the live clock on the left and, on the right, the title, category, how to read it, and an explanation of the mechanism or history, with close, previous/next and Escape;
- smooth performance, with off-screen clocks not drawing, and a small toolbar with grid size, pause/play and a time slider;
- clean code that runs by opening the file.

Later messages in the session asked about progress and said to go ahead and finish.

**Session 2 (16 October): to 240, and the kit.** “Attached is an HTML file containing a hundred timepieces that you prepared for me. I am very happy with the results, and I want to see what other ideas you can come up with on your own for additional timepieces. Please create a new version of this file with a total of 240 clocks. Include all 100 clocks in the current file, and create 140 more in a diverse and interesting variety of styles and types. Arrange all of the clocks in a grid as at present, but have the grid reordered randomly each time the page is loaded.” Then: “How far can you continue this? Could you increase the number of clocks to a thousand, each as distinctive and interesting as the ones so far? If so, how should I prompt you — in this same conversation, or with a prompt that I give you at the beginning of a new conversation?” After Claude recommended a fresh conversation for each round, with a reusable kit: “Yes, please give me that kit and starter prompt, as well as the memory fix to the gallery.”

**Session 3 (16 October): to 380.** The starter prompt from the kit, with the gallery and the kit attached:

> Attached are my clock gallery (an HTML file) and its build kit (clock-kit.zip). Please run another expansion round, following the kit’s ORCHESTRATION.md.
>
> Add 140 new clocks (7 batches of 20), each as distinctive, beautiful and carefully made as the existing ones, with none repeating a concept, mechanism or look already in REGISTER.md. Draw on IDEAS.md and your own ideas.
>
> Work in orchestration mode: make the design decisions yourself without checking with me, and tell me at the end anything I should review.
>
> Test everything as the playbook describes, and send back weak clocks for revision before assembling.
>
> Return the updated gallery HTML (renamed for the new count) and an updated clock-kit.zip with the register, ideas, changelog and any improved tools.

**Session 4 (17–18 October): to 500.** The same starter prompt, changed to “Add 120 new clocks (6 batches of 20) … to bring the total number of clocks to 500”, with one addition: “For this round, create at least one batch of 20 that have abstract and/or geometric themes.”

### Time and tokens

The figures below are estimates. Times are wall-clock hours from Tom’s first prompt to delivery; Tom was not present for most of that time. Subagent tokens are the total tokens processed by the parallel builders. Rounds 2–4 were reported at the time; session 1 was not recorded and is estimated from the work done. Each session also used a few hundred thousand tokens in the orchestrating conversation itself, which was not measured.

| Session              | Result                                         | Subagents                                   | Time                                                            | Subagent tokens                                                                          |
|----------------------|------------------------------------------------|---------------------------------------------|-----------------------------------------------------------------|------------------------------------------------------------------------------------------|
| September experiment | 100 clocks (separate code)                     | none                                        | about 2 hours (estimate)                                        | none; under 1 million in the conversation (estimate)                                     |
| 1 · 15 Oct           | 100 clocks, single file, about 1 MB            | 15 runs (several interrupted)               | about 8 hours, including about 3 hours lost to interrupted runs | about 2–3 million (estimate)                                                             |
| 2 · 16 Oct           | 240 clocks, memory fix, clock kit              | 7 builders + 1 kit test                     | about 3½ hours                                                  | about 3 million                                                                          |
| 3 · 16 Oct           | 380 clocks, 4.8 MB                             | 10 runs (7 batches; 2 failed runs replaced) | about 4¼ hours                                                  | about 5 million                                                                          |
| 4 · 17–18 Oct        | 500 clocks, 6.3 MB, category filter and search | 6 builders                                  | about 2¼ hours                                                  | about 3.5 million                                                                        |
| **Total**            |                                                |                                             | **about 18 hours**                                              | **about 14–15 million**, roughly 15–17 million including the orchestrating conversations |

For the final round alone (session 4), the six builders took 55–76 minutes each and used 490,000–670,000 tokens each. Fourteen of the 120 new clocks were sent back for revision, mostly because they were hard to read at gallery size.

## The build kit

`clock-kit.zip` in this repository holds everything needed to add more clocks in a new conversation with Claude:

| File                | Purpose                                                                                                                                                                                                                                  |
|---------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `STARTER_PROMPT.md` | The prompt to paste at the start of a round, with the gallery HTML and the kit attached                                                                                                                                                  |
| `ORCHESTRATION.md`  | Claude’s step-by-step playbook for a round, with benchmarks and lessons from earlier rounds                                                                                                                                              |
| `BRIEF.md`          | The specification given to each builder: the clock API, quality bar, performance budget and testing procedure                                                                                                                            |
| `REGISTER.md`       | All 500 clocks by category, with a one-line description of each, to avoid repeats                                                                                                                                                        |
| `IDEAS.md`          | About 165 unbuilt ideas for future clocks                                                                                                                                                                                                |
| `examples/`         | The assignment lists from rounds 2–4                                                                                                                                                                                                     |
| `tools/`            | Scripts to split the HTML into parts and reassemble it, test clocks in headless Chromium, render snapshots and contact sheets, test the whole page, measure frame rate, check non-Latin text for corruption, and regenerate the register |

The page can be split into editable source files and rebuilt exactly:

```
python3 tools/split.py five-hundred-clocks.html     # writes head.html, core.js, parts/part*.js, app.js, tail.html
python3 tools/assemble.py --auto                    # rebuilds the page and names it after the clock count
```

The kit’s `README.md` has a changelog of every round.

## Known limitations

- Testing was done in headless Chromium on Linux. Fonts differ between systems, so some text-based clocks will look slightly different on macOS, Windows and mobile devices.
- A few of the original clocks are heavier to draw than the rest; on slow machines the gallery runs at 30 redraws per second.
- Some clocks use reading conventions invented for the gallery (for example, sending the time over a historical telegraph code). Each is described in that clock’s “How to read it” panel.
