# Phierce Web

Open-source Python tooling for LLM applications, built by [Mike Farr](https://www.linkedin.com/in/mike-farr-slc/).

Several projects on one foundation. **pf-core** is that foundation; **pagespeak**, **pagespring**, **deckwright**, **x32scene**, **logicxkit** and **groovebin** are built on it. More are in the works.

```mermaid
flowchart LR
    A["Documentation sites<br/>help centers, API specs"] --> B["pagespring<br/><i>acquire + normalize</i>"]
    B --> C["pagespeak<br/><i>structure + split</i>"]
    D["PDF, Word, Office<br/>HTML, Markdown"] --> C
    C --> E["Retrieval-ready<br/>markdown corpus"]
    F["pf-core — LLM clients, versioned prompts, cost tracking, jobs"] -.-> B
    F -.-> C
    F -.-> G["deckwright<br/><i>deck spec → branded .pptx</i>"]
    F -.-> H["x32scene<br/><i>plain English → console scene</i>"]
    F -.-> I["logicxkit<br/><i>read + edit Logic sessions</i>"]
    F -.-> J["groovebin<br/><i>drum maps + MIDI pattern library</i>"]
```

---

### [pf-core](https://github.com/phierceweb/pf-core) · `pip install pf-core`

A Python foundation for building LLM applications whose prompts and spend you can actually see. Every prompt lives in versioned configuration, and every call is recorded — prompt version, model, provider, response, tokens, cost — and replayable, in SQLite, MySQL, or Postgres.

The landscape solves each of those concerns separately: one product routes calls across providers, another traces them, another versions prompts, another runs evals. Stitch several together and no shared key joins a prompt version to the tokens it burned or the validation it failed. pf-core is those concerns designed as one system.

Budgets are checked before the call and a cache refuses to pay twice for identical work. Approved runs promote into a golden set that a new model or prompt replays against before it ships. Long batch jobs resume after a crash instead of starting over. One interface covers OpenRouter, the Anthropic SDK, and Claude Code, so a batch can route onto a local Claude Max session at no per-call cost and still be tracked the same way.

The base install is dependency-light; the LLM, database, and web tiers install independently of each other.

### [pagespeak](https://github.com/phierceweb/pagespeak) · `pip install pagespeak`

Converts documents into small, retrievable markdown that stays intelligible to an LLM at minimal token cost.

The hard part isn't conversion, it's structure. pagespeak uses existing converters — Marker and Docling for PDFs, others for non-PDF formats — and mechanically repairs the heading corruptions each one is known to produce. It can blend them, too: Marker produces deeper hierarchy, Docling cleaner tables.

Sections are cut on hierarchy rather than size, and each chunk carries breadcrumbs back to its parent sections, so the model can tell where a chunk sits and not just what it says. Images can optionally go to a vision model, which decides whether one is a diagram worth converting to Mermaid.

It's the ingestion and structuring stage of a retrieval pipeline and stops there — no embeddings, no vector storage, no query layer. Pair it with your own.

### [pagespring](https://github.com/phierceweb/pagespring) · `pip install pagespring`

The acquisition front end to pagespeak. Point it at a manual's URL: a pattern recognizes the source type, acquires the raw pages, and normalizes them into one clean file with absolute asset URLs. Converting the result is pagespeak's job.

For publicly available documentation only — vendor manuals, help centers, open textbooks, API specs. No login handling, no paywall traversal, no bot-detection evasion. It identifies itself, honors `429 Retry-After`, backs off on server errors, paces its requests, and caps crawl size. Closer to "Save Page As" than to an autonomous crawler.

### [deckwright](https://github.com/phierceweb/deckwright) · `git clone`

Tell an AI agent your story; get a branded PowerPoint deck.

Deciding what goes on the slides should be the hard part, not the slide application. A deck is a `.deck.yaml` — one document per slide — and the design lives in the theme rather than in the deck: type, colour, spacing and the grid, decided once and applied everywhere. Point a deck at a real PowerPoint template and `conform` builds a theme from it, so the same deck comes out in that brand.

That is what makes iterating cheap. Add a slide, cut two, reorder the middle, turn that list into a column chart — each of those is one sentence to an assistant that has the spec open, and the deck rebuilds in seconds. The story is what you're editing, not the program. The repo ships three reference docs the agent reads while it builds, covering the design calls it would otherwise guess at: which chart the numbers want and whether they want one at all, what shape a slide that isn't a chart should take, and whether the arrows in a diagram are making a claim. Whatever it still gets wrong, the compiler catches before it writes the file — text that doesn't fit stops the build with a message naming the fix.

### [x32scene](https://github.com/phierceweb/x32scene) · `pip install x32scene`

AI-assisted control of a Behringer X32 or M32 mixing console, through the files the console itself writes. Tell an assistant what the night needs: this band's inputs moved to the second stage box, that player's in-ear mix carried over from last week, a plate on FX 4 at 2.1 seconds. You get back a scene you load on the desk. Every command has a `--json` form and can list the vocabulary the console accepts, so an agent can drive the whole surface as well as a person can.

That works because the console keeps a whole show as a few thousand lines of plain text: routing, head amps, EQ and dynamics on every channel, every monitor mix, the effects rack. x32scene reads and writes those files directly, including scenes, snippets, channel, effect and routing presets, and shows. Editing through the desk is time-consuming and difficult, changing one thing at a time. **x32scene** can change entire swaths of functionality at once.

Parsing is byte-faithful, so an edited file differs from its source by exactly the lines that were meant to change and nothing else. `diff` reports what moved in the desk's own terms, values are checked against the console's vocabulary before anything is written, and a plan for a whole night is refused if the result strays outside the paths the plan named.

### [logicxkit](https://github.com/phierceweb/logicxkit) · `pip install logicxkit`

A Logic Pro session is an opaque binary. Which plugin sits in which slot, what that plugin actually saved, the fader, the routing, the track list, the groups — all of it is reachable only by opening Logic and looking at it. There is no way to diff two sessions, script one change across twenty of them, or read what a third-party plugin stored inside a strip you saved last year.

logicxkit reads those containers directly. Inventory a session, diff two of them, or diff one against your channel-strip library to find the channel that quietly stopped matching the strip it names. Copy a control bar or a set of mixer groups from one project onto another, migrate an old song onto a newer template, repoint strip references after a library rename. `au strip` decodes the third-party plugin state embedded in a session — the layer Logic reports as a preset name and nothing more.

None of these formats are documented, so all of it came out of measurement: one deliberate change per Logic save, then a byte diff against the save before it. The control bar is the tidiest example — every button id was pinned across fifty single-toggle saves, and a bar written by the tool and copied whole onto another project came up in Logic with that exact set. That standard is not uniform across the tool, so a generated table records what each command was measured against and when, and any write that has not been confirmed in Logic says so before it runs.

macOS only, since it reads and writes Logic's own files. Every project write lands on a copy — the input session is never modified.

### [groovebin](https://github.com/phierceweb/groovebin) · `pip install groovebin`

A drum part is written for one kit's note numbers. Play it through a different drum instrument and the notes land on the wrong sounds — a rimshot arrives as a cowbell, a cymbal choke stops nothing. The usual fix is to open the file in a DAW and move notes by hand, one at a time.

groovebin translates the part instead. It reads and writes Standard MIDI Files, moves notes between drum maps — General MIDI, Addictive Drums 2, Logic's Drum Kit Designer — transforms them by selection (velocity curves, humanize, swing, note lengths), and indexes a folder of patterns so a groove can be found by role, meter and tempo instead of by opening files until the right one turns up. Nothing needs a DAW running or a plug-in installed. logicxkit uses it for the MIDI it reads out of and writes into Logic projects.

The note numbers are the whole claim, so every note in a packaged map carries its source: General MIDI from the MIDI Manufacturers Association's own percussion table (47 notes), Addictive Drums 2 from the keymap its vendor publishes (80), and Drum Kit Designer from Apple's published figure, its octave reading corroborated against the pitches counted in four exported drummer regions (30). A note no source names stays out of the map. A note with no counterpart in the destination is reported rather than approximated onto a neighbour, and a choke never falls back onto a strike. Reading a file and writing it back keeps every note and every other event at its tick, so what comes back differs from the source only where the translation touched it.

It ships note-number tables only — no sounds, patterns or MIDI content from any vendor — and it does no audio analysis: no onsets, no transients, no flex markers. Alpha, so the commands and the index format may still change before 1.0.

---

All Apache-2.0 licensed, except pf-core and pagespeak, which are MIT. (deckwright also bundles the Material icon set, under Apache-2.0.)

### About

I'm a software architect and engineering manager. These are built nights and weekends. I hope others find these tools useful. I welcome collaboration. 

[LinkedIn](https://www.linkedin.com/in/mike-farr-slc/)
