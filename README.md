# random-translation

**A small, portable utility that plucks out language translations already on your system.**

More specifically, it scans the system for GNU gettext `.mo` files and emits translations from a
randomly picked translation table for one of the specified languages.

```
┌───=[ me :: laptop ]-( 0 )-[ ~ ]
└──( random-translation pt -n 3
  App: Systemd (Portuguese)

  en  Authentication is required to get system description.
  pt  É necessária autenticação para obter a descrição do sistema.

  en  Allow applications to inhibit system handling of the suspend key
  pt  Permitir que aplicações inibam o sistema de gerir o botão de suspensão

  en  Authentication is required to change the password of a user's home area.
  pt  É necessária autenticação para alterar a palavra-passe da área home de um utilizador.
```

## Who is this for?

**Developers and translators** — if you work on software that ships translations, this tool lets you
browse your actual installed `.mo` files as human-readable text. Quickly see what strings your app
exposes, sanity-check translations against their English originals, or audit what gettext catalogs
are installed on a given system. No Python, no gettext toolchain required — it reads the binary `.mo`
format directly.

**Language learners** — every string is shown side-by-side with its English original, drawn from
real software your machine already uses. Run it a few times and you get a natural sampling of
everyday UI language: button labels, error messages, confirmation dialogs — the vocabulary that
actually appears on screens.

**Screensaver enthusiasts** — pair it with [Phosphor](https://www.jwz.org/xscreensaver/), the
retro terminal-style XScreenSaver, and your idle screen becomes a slow river of bilingual text.
Pick one language or mix several; set `-n 3` for dense bursts or `-n 1` for sparse, contemplative
lines. It's surprisingly meditative.

**Anyone who didn't know they wanted this** — your computer already contains tens of thousands of
translated strings from hundreds of applications. This tool is a window into that corpus. Run
`random-translation es` and watch Spanish UI phrases scroll by. It's oddly compelling.

## Features

- **Default behavior:** deep index build on first run, index-based lookup thereafter
- **Manual strategies:** `auto`, `quick`, `host-indexed`, `targeted`, `deep`, `index-only`
- **Index control:** `read`, `write`, `none` with TTL and `--refresh`
- **Output formats:** `human` (default), `raw`, `json`, `yaml`, `filename`
- **Color themes:** `jungle` (default), `aurora`, `phosphor`, `cyberpunk`, `neon`, `sakura`, `classic`, `tokyonight`, `ocean`, `mars`
- **Sampling:** print `-n N` random translated strings (great for screensavers)
- **Locale matching:** `es`, `es_ES`, `es-ES`, `es_ES.UTF-8`
- **Fallbacks:** uses `msgunfmt` when available; pure `od`/`awk` otherwise
- **Explicit reader:** `--reader auto|msgunfmt|od` to pin or test a specific extraction backend
- **Direct file input:** `--file FILE` to read any `.mo` file directly, bypassing the index and discovery
- **Debugging:** `--debug` shows discovery and index-mode decisions
- **Modern XDG support:** honors `XDG_CACHE_HOME` (cache location) and `XDG_DATA_HOME/locale`
  (per-user locale data) when present; defaults to sensible platform locations otherwise
- **Filtering:** opt-in `--filter` to restrict discovery to prefix paths (see "Filtering and deep scans")

## Install

```bash
# Preferred: install sets owner and mode in one step
sudo install -m 755 random-translation /usr/local/bin/

# Or copy manually
sudo cp random-translation /usr/local/bin/
sudo chown root:root /usr/local/bin/random-translation
sudo chmod 755 /usr/local/bin/random-translation
```

After installing, build the index once so it's ready for use:

```bash
random-translation --index-init
```

This scans your system for `.mo` files and caches the results. Subsequent runs are near-instant.
Automated installers (packages, dotfile scripts, etc.) can run this as a post-install step.

## Quickstart

Pick a random Greek translation. If no index exists yet, this also builds it:

```bash
random-translation el
```

Get 5 random translated strings:

```bash
random-translation -n 5 es
```

Mix multiple languages in one run:

```bash
random-translation es fr de
```

Emit JSON or YAML for programmatic use:

```bash
random-translation es --output json
random-translation es --output yaml
```

Include PO metadata (file header and msgctxt) in output:

```bash
random-translation es --meta
```

Show the path that would be read without reading it:

```bash
random-translation es --dry-run
```

Read a specific `.mo` file directly:

```bash
random-translation --file /usr/share/locale/fr/LC_MESSAGES/wget.mo
random-translation --file /path/to/app.mo --output json -n 5
```

Force a specific extraction backend (useful for testing or non-UTF-8 files):

```bash
# Always use the built-in od/awk parser (no msgunfmt dependency)
random-translation es --reader od

# Require msgunfmt explicitly
random-translation es --reader msgunfmt
```

Target a specific set of roots:

```bash
random-translation fr --strategy targeted --roots "/usr/share/locale:/opt/myapp/share/locale"
```

## Default behavior

Running `random-translation LANGS...` with no index flags is the recommended path. On the
first run it performs a deep scan, writes the master index, and builds per-language indexes. On
later runs it reuses that index automatically for near-instant selection.

XDG notes: the tool respects modern XDG environment variables. By default the index is
stored under `$XDG_CACHE_HOME/random-translation` (or `~/.cache/random-translation`), and
per-user locale data directories under `$XDG_DATA_HOME/locale` (or `~/.local/share/locale`)
are included in discovery. On macOS the index defaults to `~/Library/Caches` when XDG
variables are not set.

### WSL (Windows Subsystem for Linux)

On WSL, the `deep` strategy crosses DrvFs mounts under `/mnt` (your Windows drives), which
are orders of magnitude slower than the native Linux filesystem. The tool detects WSL
automatically and defaults to the `quick` strategy on first run instead of `deep`. This
finishes in seconds rather than minutes, at the cost of potentially missing `.mo` files
installed in unusual locations.

If you want a thorough scan on WSL, run it explicitly:

```bash
# Scan only the native Linux filesystem, exclude /mnt
random-translation --index-init --strategy deep

# Or scan specific known locations
random-translation --index-init --strategy targeted --roots "/usr/share/locale:/usr/local/share/locale"
```

### Filtering and deep scans

By default the `deep` discovery strategy performs an unrestricted scan of the filesystem
and does not implicitly restrict results to a small set of prefix roots. This avoids
accidentally omitting translations that live outside common system directories.

On Linux, `deep` is mount-aware and locale-oriented: it scans the root filesystem plus
additional local mount points individually, does not cross filesystem boundaries, skips
known virtual or network-backed mounts, and looks specifically for GNU gettext layouts
such as `*/LC_MESSAGES/*.mo`. This keeps the scan broad enough for real locale data
without wandering through arbitrary `.mo` files elsewhere on the system.

If you want to restrict discovery to a set of trusted locations, you can:

- Use `--roots` (colon-separated) to add those roots to the built-in locale roots and
  enable prefix filtering.
- Use `--filter` (colon-separated) to set an explicit list of prefix paths to filter
  discovered .mo files against. `--filter` overrides the internal LOCALE_DIRS used for
  filtering and is useful for advanced control.

Examples:

```
# Unrestricted deep scan (default behaviour for --strategy deep):
random-translation --strategy deep --index-mode write sv,fi,pl

# Restrict discovery to two locations (enable filtering):
random-translation --strategy deep --index-mode write --roots "/usr/share/locale:/opt/myapp/share/locale" el

# Explicit filter list that overrides internal LOCALE_DIRS:
random-translation --strategy deep --index-mode write --filter "/opt/myapp/locale:/home/user/.local/share/locale" fr
```

Why restrict discovery? Reasons advanced users might want to:

- Limit runtime and I/O during a deep scan on systems with very large filesystems.
- Ensure only specific application or vendor locale directories are considered.
- Avoid picking translations from transient mount points or network shares.
- Reproduce a narrow, deterministic dataset for testing.

When `--debug` is enabled, deep scans also print progress to `stderr` showing which root
or top-level path is currently being scanned.

Note: filtering is opt-in. If you previously relied on `--index-mode none` to avoid
filtering side-effects, the default deep scan now behaves the same (unrestricted).

## Manual index control

If you want to manage index behavior yourself, the manual modes are also available. `deep`
is the most thorough; `host-indexed` uses `mdfind` (macOS) or `locate` (Linux) and is faster
but may miss some locations.

Index-mode summary: `read` reuses the index (fails if missing), `write` rebuilds it and respects
`--refresh`, and `none` performs live discovery without touching the index. Use `--refresh` with
`read`/`write` to force a rebuild even before TTL expires. `--index-init` runs an index build
(defaulting to `deep`, or `quick` on WSL), prints stats, and exits without picking a translation.
You can combine it with `--strategy` to choose the scan method explicitly.

```bash
# Force a full rebuild
random-translation --strategy deep --index-mode write --refresh el tr es hu

# Rebuild using the host's file indexer
random-translation --strategy host-indexed --index-mode write --refresh el tr es hu

# Read from the existing index only (no discovery)
random-translation --strategy index-only el tr es hu
```

## Color themes

The `--theme` option selects a color palette for terminal output. Pass `--theme` with no value
to list all available themes.

| Theme        | Character                                                        |
|--------------|------------------------------------------------------------------|
| `jungle`     | Lush greens, fern, moss, canopy lime — the default               |
| `aurora`     | Aurora borealis greens, violet, sky blue                         |
| `phosphor`   | Deep electric blues and phosphorescent glow — retro CRT feel     |
| `cyberpunk`  | Blazing neon pinks, electric cyans, vivid blues                  |
| `neon`       | Brighter and bolder, maximum 256-color saturation                |
| `sakura`     | Soft pinks and warm rose tones                                   |
| `classic`    | Traditional terminal greens and ambers                           |
| `tokyonight` | Cool blues, muted greens, orange accents                         |
| `ocean`      | Deep teals, aqua, seafoam, steel blue — calm and clear           |
| `mars`       | Mars reds, rust, dusty orange — the red planet                   |

```bash
random-translation es --theme cyberpunk
random-translation ja --theme tokyonight -n 5
```

Themes have no effect when color is disabled (`--color never` or a non-color terminal).

## Using with Phosphor and XScreenSaver

[Phosphor](https://man.archlinux.org/man/extra/xscreensaver/phosphor.6.en) is a retro
terminal-style screensaver that can display text produced by a shell command. Different
builds and front ends expose the command field under different names:

- On Linux it is usually labeled **Program** in the Phosphor settings dialog.
- On macOS (e.g. `brew install xscreensaver`) the field is usually labeled **Shell Cmd**
  under Display Text in the Phosphor settings dialog.

### PATH note

XScreenSaver runs with a minimal environment and typically does not include custom PATH
entries. On Linux, `/usr/local/bin` is usually already in the default PATH so the bare
`random-translation` command works. On macOS it is safer to use the absolute path:

```
/usr/local/bin/random-translation de
```

### Character set note

Phosphor renders output via a fixed bitmap font and does not support non-Latin scripts.
Languages that use Cyrillic, Greek, Arabic, CJK, or other non-Latin scripts — `bg`, `el`,
`ru`, `ar`, `ja`, etc. — will appear as upside-down question marks. This is a Phosphor
limitation, not a bug in this tool.

**Stick to languages that use the Latin alphabet**, including those with diacritics and
accented characters: `es`, `fr`, `de`, `pt`, `it`, `pl`, `cs`, `ro`, `hu`, `fi`, `sv`,
`no`, `da`, `nl`, `tr`, and similar.

### Build the index first

Phosphor runs the command in a minimal environment with no interactive terminal, so there is
no opportunity to wait for a slow first-run index build. Run this once in a terminal before
configuring Phosphor:

```bash
random-translation --index-init
```

The index is written to your cache directory and reused on every subsequent run, including
from the screensaver.

### One-shot command

Paste directly into Phosphor's Shell Cmd or Program field:

```
/usr/local/bin/random-translation tr,es,hu,pl
```

On Linux you can use just:

```
random-translation tr,es,hu,pl
```

### Pipe mode

If your Phosphor supports pipe mode, run it with `--pipe` and `--program`:

```
phosphor --scale 4 --delay 80000 --pipe --program '/usr/local/bin/random-translation -n 1 tr es hu pl'
```

### Recommended Phosphor options

- `--scale 2` or `--scale 4` to adjust pixel size
- `--delay 80000` to slow the character reveal for readability
- `--pipe` when feeding a continuous stream

## Tips for screensaver use and language learning

- **Run `--index-init` in a terminal first** so the index is ready before the screensaver needs it.
- **Use `--strategy index-only`** when you want to guarantee no discovery occurs at screensaver time.
- **Use `-n 1`** for single-line displays; use `-n N` for multi-line bursts.
- **Mix languages** on the command line to rotate phrases from multiple targets.
- **Use `--roots` + `--strategy targeted`** to include app-specific locale directories.
- **Adjust Phosphor `--delay` and `--scale`** to tune pacing and readability.
- **Log selections** if you want to review phrases later (wrap the command and append to a file).

## Troubleshooting

**`No .mo files found`**

- Let auto mode build the index: `random-translation LANG`
- Or force a rebuild: `random-translation --strategy deep --index-mode write --refresh LANG`
- Use `--roots` with `--strategy targeted` to point at known locale directories
- Use `--debug` to inspect discovery and index-mode decisions
- If translations are unexpectedly missing, check whether `--roots` or `--filter` were used
  (these enable prefix filtering), and verify XDG variables (`XDG_DATA_HOME`, `XDG_CACHE_HOME`).

**JSON/YAML output fails or is empty**

This can happen with very old `msgunfmt` versions (e.g. gettext 1.0 on macOS Big Sur) that
produce no output for some files. The tool automatically falls back to the `od`/`awk` path
in that case. If you still get no output, try pinning the parser explicitly:

```bash
random-translation es --reader od
```

**Non-UTF-8 files show garbled output**

Some `.mo` files use legacy encodings (CP1251, Latin-1, etc.) rather than UTF-8. Both
`msgunfmt` and the `od`/`awk` path emit the raw bytes as-is; the garbling is in the
terminal rendering, not the tool. The content is correct — your terminal just needs to
match the file encoding. You can check the encoding in the PO header with `--reader od`
and `--output human` to see the `Content-Type` header entry.

**Install `msgunfmt` for better compatibility**

`brew install gettext` on macOS, `sudo pacman -S gettext` on Arch, `sudo apt install gettext`
on Debian/Ubuntu. The `od`/`awk` fallback works without it, but `msgunfmt` handles more
edge cases in the PO format.

**Screensaver shows nothing**

Check that the index exists and is non-empty: `random-translation --index-stats`. If the
index is stale or empty, rebuild it in a terminal.

## Reference

```
Usage: random-translation [options] LANGS...

  LANGS may be one or more language codes (space or comma separated).

  Examples:
    - Single codes: es tr el
    - Comma list: "es,el,tr"
    - Locale variant: es_ES

Options
  --roots PATHS       Colon-separated roots to search (overrides default roots)
  --filter PATHS      Colon-separated prefix paths to restrict discovery (overrides default LOCALE_DIRS and enables filtering)
  --strategy STR      Discovery strategy: auto|quick|host-indexed|targeted|deep|index-only
  --index-mode MODE   Index behavior: read|write|none
  --refresh           Force rebuild of the .mo list (applies when index-mode is write or when using transient list)
  --ttl SECS          Index TTL in seconds (default 3600)
  --debug             Print debug information showing what each strategy and index-mode decision does
  --meta              Include PO metadata (file header + msgctxt) in output
  --color WHEN        Colorize output: auto|always|never (default: auto)
  --no-color          Alias for --color never
  --theme THEME       Color theme: jungle|aurora|phosphor|cyberpunk|neon|sakura|classic|tokyonight|ocean|mars (default: jungle)
  -n N                Print N random msgstr strings only (useful for screensavers)
  --output MODE       Output mode: human|raw|json|yaml|filename  (default: human)
  --index-stats       Print index stats (entries, age, size) then exit
  --index-init        Build the index and exit (default: deep scan; quick on WSL; override with --strategy)
  --version           Print version and exit
  --reader MODE       Extraction backend: auto|msgunfmt|od  (default: auto)
  --file FILE         Read directly from this .mo file (bypasses index and discovery)
  --dry-run           Print the path that would be read and exit (alias for --output filename)
  -h, --help          Show this help

Exit codes
  0  success
  1  usage / argument error
  2  missing tools for extraction
  3  no matches found
```

## Contributing

Fork, add tests or improvements, and open a pull request. Keep changes portable and avoid
platform-specific assumptions without guards.

## License

ISC
