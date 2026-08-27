<div align="center">

# AtlasCLI

**The Atlas knowledge base in your terminal - 3,000+ IT topics and definitions, fetched once and then queried offline, with no dependencies beyond Python itself.**

[![Python](https://img.shields.io/badge/python-3.8+-blue?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![Dependencies](https://img.shields.io/badge/dependencies-none-brightgreen?style=flat-square)](https://docs.python.org/3/library/)
[![Atlas](https://img.shields.io/badge/atlas-drew--gnr.xyz-ff4d2e?style=flat-square)](https://drew-gnr.xyz/atlas/)
[![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)](LICENSE)

</div>

---

## Overview

**Run it with one command: `atlas <thing>`.** Atlas already exists as a
website with 816 topics across 34 sections; this puts the same content
behind a single command so you can check something without opening a
browser, breaking flow, or leaving an SSH session.

It reads the exact same generated bank the website uses
(`atlas/bank.js`), so there is no second copy of the content to keep in
step. The bank is fetched once, cached locally, and everything after that
works offline.

One file, standard library only, no build step - the same way the rest of
the site is built.

## Requirements

- Python 3.8+
- Nothing else. No pip install, no virtualenv, no `requirements.txt`.

A network connection is needed for the first run only, to fetch the bank.

## Setup

```bash
chmod +x atlas
cp atlas ~/.local/bin/
```

Anywhere on `PATH` works. To run it without installing:

```bash
./atlas subnetting
```

## Usage

```bash
atlas subnetting              # look something up
atlas -c networking -r        # a random entry from one category
atlas -c linux -d 2 systemd   # filter by category and difficulty
atlas -k definition dns       # definitions only, skip the longer topics
atlas -a vlsm                 # every match in full, not just the best one
atlas -l                      # list the 34 categories with entry counts
atlas --update                # refresh the cached bank now
```

The best match prints in full; anything else found is listed underneath as
a one-liner so you can see whether you asked the wrong question.

## Options

| Flag | Description |
|---|---|
| `-c`, `--category` | Limit to one category, by id (`networking`) or name (`Networking`). Partial names work if unambiguous. |
| `-d`, `--difficulty` | `0` easy, `1` medium, `2` hard, `3` extreme. Same four tiers as the website's quiz. |
| `-k`, `--kind` | `topic` for the longer explanations, `definition` for the short ones. |
| `-n`, `--limit` | How many matches to consider. Default `6`. |
| `-r`, `--random` | One random entry, respecting whatever other filters are set. |
| `-a`, `--all` | Print every match in full rather than just the best one. |
| `-l`, `--list` | List categories with entry counts. |
| `--update` | Force a refresh of the cached bank. |
| `--source` | A different `bank.js` URL or local path. |
| `--no-color` | Disable colour. `NO_COLOR` is honoured too. |

Colour switches off automatically when output is piped, so
`atlas dns | less` stays readable.

Exit codes: `0` found something, `1` found nothing or bad input - so it
composes properly in scripts.

## Configuration

| Variable | Description |
|---|---|
| `ATLAS_BANK` | Bank source, URL or local path. Default `https://drew-gnr.xyz/atlas/bank.js`. |
| `NO_COLOR` | Set to anything to disable colour output. |
| `XDG_CACHE_HOME` | Cache location. Defaults to `~/.cache`, giving `~/.cache/atlas/bank.js`. |

Point it at a local checkout while working on Atlas itself:

```bash
export ATLAS_BANK=~/site/atlas/bank.js
```

The cache is refetched when it is more than a week old, or on `--update`.
If the network is unreachable but a cache exists, it warns and uses the
cache rather than failing. If the cache is corrupt it refetches. `bank.js`
is generated and marked "do not edit by hand", so this only ever reads it.

## How matching works

Ranked in tiers, strongest first:

| Tier | Match |
|---|---|
| 1 | Title matches the query exactly |
| 2 | Title starts with the query |
| 3 | Title contains the query |
| 4 | Same source page - the query appears in the entry's slug |
| 5 | Fuzzy title similarity, at or above 0.70 |
| 6 | The query appears in the body text |

Every tier below the third carries a similarity tiebreaker. Without one,
whole groups tie at a single score: searching `subnetting` makes every
definition on the subnetting page a slug match, and they would come back
alphabetically rather than by relevance.

The 0.70 fuzzy floor is deliberate rather than tuned by feel. At 0.60 the
string `zzzznotathing` scores 0.609 against "do nothing" and the tool
returns a confident wrong answer with a success exit code.

## Tech

- Single file, ~420 lines, standard library only - `argparse`, `difflib`,
  `json`, `urllib`, `textwrap`.
- `bank.js` is `window.ATLAS_BANK = {...};`, so the wrapper is stripped and
  the rest parsed as JSON.
- Colours are the site palette from `Assets/style.css`, and the difficulty
  tints come from `atlas/study.js`, so Easy is phosphor green and Extreme
  is signal orange exactly as on the website.
- Truecolor ANSI, suppressed when stdout is not a TTY.

## Roadmap

- Quiz mode, difficulty-filtered off the `d` field
- Flashcard mode using `m`, which already holds each definition with the
  term masked out - the cloze work is done, it just needs rendering
- Spaced repetition, mirroring `atlas/store.js` but writing to
  `~/.local/share/atlas/` instead of localStorage
- Shell completion for category names

## License

MIT
