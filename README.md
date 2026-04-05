# random-translation

**A small, portable tool to pick a random installed GNU gettext `.mo` file for one or more
languages and emit translations.**

Ideal for testing, demos, language learning, and screensavers. Portable, blazing fast when cached,
and works on macOS, Linux, and other Unix-like systems.

```
┌───=[ me :: station ]-( 0 )-[ ~ ]
└──( random-translation pt -n 3 --output json
[
  {
    "msgid": "An extension was expected but was not seen",
    "msgstr": "Uma extensão era esperada, mas não foi vista"
  },
  {
    "msgid": "A packet with illegal or unsupported version was received.",
    "msgstr": "Um pacote com versão ilegal ou sem suporte foi recebido."
  },
  {
    "msgid": "No supported cipher suites have been found.",
    "msgstr": "Nenhum conjunto de cifras com suporte foi localizado."
  }
]
```

## Features

- **Default behavior:** deep cache build on first run, cached lookup thereafter
- **Manual strategies:** `auto`, `quick`, `indexed`, `targeted`, `deep`, `cached`
- **Cache control:** `read`, `write`, `none` with TTL and `--refresh`
- **Output formats:** `normal` (PO-like), `json`, `yaml`, `filename`
- **Readable output:** optional ANSI colors, optional PO header (`-H`), optional metadata (`--meta`)
- **Sampling:** print `-n N` random translated strings (great for screensavers)
- **Locale matching:** `es`, `es_ES`, `es-ES`, `es_ES.UTF-8`
- **Fallbacks:** uses `python3` or `msgunfmt` when available; pure `od`/`awk` otherwise
- **Debugging:** `--debug` shows discovery and cache decisions
- **Modern XDG support:** honors `XDG_CACHE_HOME` (cache location) and `XDG_DATA_HOME/locale`
  (per-user locale data) when present; defaults to sensible platform locations otherwise

## Install

```bash
# Preferred: install sets owner and mode in one step
sudo install -m 755 random-translation /usr/local/bin/

# Or copy manually
sudo cp random-translation /usr/local/bin/
sudo chown root:root /usr/local/bin/random-translation
sudo chmod 755 /usr/local/bin/random-translation
```

## Default behavior

Running `random-translation LANGS...` with no cache flags is the recommended path. On the
first run it performs a deep scan, writes the cache, and builds per-language indexes. On
later runs it reuses that cache automatically for near-instant selection.

XDG notes: the tool respects modern XDG environment variables. By default the cache
is placed under `$XDG_CACHE_HOME/random-translation` (or `~/.cache/random-translation`), and
per-user locale data directories under `$XDG_DATA_HOME/locale` (or `~/.local/share/locale`)
are included in discovery. On macOS the cache defaults to `~/Library/Caches` when XDG
variables are not set.

```bash
# First run: deep scan + cache build
random-translation sv,fi,pl

# Later runs: cached automatically
random-translation sv,fi,pl
```

## Manual cache control

If you want to manage cache behavior yourself, the manual modes are also available. `deep`
is the most thorough; `indexed` uses `mdfind` (macOS) or `locate` (Linux) and is faster but
may miss some locations.

Cache-mode summary: `read` reuses the cache (fails if missing), `write` rebuilds it and respects
`--refresh`, and `none` performs live discovery. Use `--refresh` with `read`/`write` to force a
rebuild even before TTL expires. `--cache-init` runs `deep` + `write`, prints stats, and exits
without picking a translation.

```bash
# Force a full rebuild
random-translation --strategy deep --cache-mode write --refresh el tr es hu

# Rebuild using the system indexer
random-translation --strategy indexed --cache-mode write --refresh el tr es hu

# Use only the existing cache
random-translation --strategy cached el tr es hu
```

## Quickstart

Pick a random Greek translation and print PO-like output. If no cache exists yet, this also builds it:

```bash
random-translation el
```

Get 5 random translated strings:

```bash
random-translation -n 5 es
```

Emit JSON for programmatic use:

```bash
random-translation es --output json
```

Include header metadata in structured output:

```bash
random-translation es --output json --header
random-translation es --output yaml --header
```

Show context metadata in normal PO-like output:

```bash
random-translation es --meta
```

Show the path that would be read without reading it:

```bash
random-translation es --dry-run
```

Target a specific set of roots:

```bash
random-translation fr --strategy targeted --roots "/usr/share/locale:/opt/myapp/share/locale"
```

## Using with Phosphor and XScreenSaver

[Phosphor](https://man.archlinux.org/man/extra/xscreensaver/phosphor.6.en) is a retro
terminal-style screensaver that can display text produced by a shell command. Different
builds and front ends expose the command field under different names:

- On macOS (e.g. `brew install xscreensaver`) the field is usually labeled **Shell Cmd**
  under Display Text in the Phosphor settings dialog.
- On Linux and many XScreenSaver front ends it is usually labeled **Program**.

### PATH note

XScreenSaver runs with a minimal environment and typically does not include `/usr/local/bin`
in `PATH`. On Linux that directory is usually already in the default PATH, so the bare
`random-translation` works; on macOS it’s easiest to point Phosphor at the absolute path:

```
/usr/local/bin/random-translation de
```

### macOS sandbox note

On macOS, `/usr/bin/python3` is an Xcode Command Line Tools stub that invokes `xcrun`,
which is blocked inside the App Sandbox that XScreenSaver/Phosphor runs in. The script
automatically detects and skips this stub, preferring a real interpreter (e.g. from
Homebrew at `/opt/homebrew/bin/python3` or `/usr/local/bin/python3`). If no real Python
is found it falls back to a pure `od`/`awk` `.mo` reader that works without any external
tools. Run the command once in a terminal first so the cache is ready before screensaver
time.

### One-shot command

Paste directly into Phosphor's Shell Cmd or Program field:

```
/usr/local/bin/random-translation el,tr,es,hu
```

### Pipe mode

If your Phosphor supports pipe mode, run it with `--pipe` and `--program`:

```
phosphor --scale 4 --delay 80000 --pipe --program '/usr/local/bin/random-translation -n 1 el tr es hu'
```

### Recommended Phosphor options

- `--scale 2` or `--scale 4` to adjust pixel size
- `--delay 80000` to slow the character reveal for readability
- `--pipe` when feeding a continuous stream

## Tips for screensaver use and language learning

- **Run it once in a terminal first** so the automatic cache is ready for later use.
- **Use `--strategy cached`** when you want to guarantee cache-only behavior.
- **Use `-n 1`** for single-line displays; use `-n N` for multi-line bursts.
- **Mix languages** on the command line to rotate phrases from multiple targets.
- **Use `--roots` + `--strategy targeted`** to include app-specific locale directories.
- **Adjust Phosphor `--delay` and `--scale`** to tune pacing and readability.
- **Log selections** if you want to review phrases later (wrap the command and append to a file).

## Troubleshooting

**`No .mo files found`**

- Let auto mode build the cache: `random-translation LANG`
- Or force a rebuild: `random-translation --strategy deep --cache-mode write --refresh LANG`
- Use `--roots` with `--strategy targeted` to point at known locale directories
- Use `--debug` to inspect discovery and cache decisions

**`xcrun: error: cannot be used within an App Sandbox`**

This is the macOS Xcode stub being invoked as `python3`. Install Homebrew Python
(`brew install python3`) so the script can find a real interpreter, or rely on the
built-in `od`/`awk` fallback which requires no Python at all.

**JSON/YAML output fails**

Ensure `python3` (Homebrew recommended on macOS) or `msgunfmt` (`brew install gettext`) is
installed. The `od`/`awk` fallback only covers `-n N` output, not JSON or YAML.

**Screensaver shows nothing**

Check that the cache exists and is non-empty: `random-translation --cache-stats`. If the
cache is stale or empty, rebuild it in a terminal.

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
  --strategy STR      Discovery strategy: auto|quick|indexed|targeted|deep|cached
  --cache-mode MODE   Cache behavior: read|write|none
  --refresh           Force rebuild of the .mo list (applies when cache-mode is write or when using transient cache)
  --ttl SECS          Cache TTL in seconds (default 3600)
  --debug             Print debug information showing what each strategy and cache decision does
  -H, --header        Include PO header metadata in output (default: hidden)
  --meta              Include metadata entries in normal output (e.g. msgctxt)
  --color WHEN        Colorize output: auto|always|never (default: auto)
  --no-color          Alias for --color never
  -n N                Print N random msgstr strings only (useful for screensavers)
  --output MODE       Output mode: normal|json|yaml|filename  (default: normal)
  --cache-stats       Print cache stats (entries, age, size) then exit
  --cache-init        Build the cache (deep scan + write) and exit
  --version           Print version and exit
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
platform-specific assumptions without guards. Consider adding new discovery strategies.

## License

ISC
