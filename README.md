# random-translation

**A small, portable tool to pick a random installed GNU gettext `.mo` file for one or more
languages and emit translations.**

Ideal for testing, demos, language learning, and screensavers. Portable, fast when cached,
and works on macOS, Linux, and other Unix-like systems.

## Features

- **Discovery strategies:** `quick`, `indexed`, `targeted`, `deep`, `cached`
- **Cache control:** `read`, `write`, `none` with TTL and `--refresh`
- **Output formats:** `normal` (PO-like), `json`, `yaml`, `filename`
- **Sampling:** print `-n N` random translated strings (great for screensavers)
- **Locale matching:** `es`, `es_ES`, `es-ES`, `es_ES.UTF-8`
- **Fallbacks:** uses `python3` or `msgunfmt` when available; pure `od`/`awk` otherwise
- **Debugging:** `--debug` shows discovery and cache decisions

## Install

```bash
# Preferred: install sets owner and mode in one step
sudo install -m 755 random-translation /usr/local/bin/

# Or copy manually
sudo cp random-translation /usr/local/bin/
sudo chown root:root /usr/local/bin/random-translation
sudo chmod 755 /usr/local/bin/random-translation
```

## Build a cache

Building a cache once makes subsequent runs fast and reliable. The `deep` strategy does a
full filesystem scan and is the most thorough; `indexed` uses `mdfind` (macOS) or `locate`
(Linux) and is faster but may miss some locations.

```bash
# Thorough: full scan (slower, recommended for first-time setup)
random-translation --strategy deep --cache-mode write --refresh el tr es hu

# Fast: uses system indexer when available
random-translation --strategy indexed --cache-mode write --refresh el tr es hu
```

After building the cache you can use `--strategy cached` for near-instant runs.

## Quickstart

Pick a random Greek translation and print PO-like output:

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

Show the path that would be read without reading it:

```bash
random-translation es --dry-run
```

Target a specific set of roots:

```bash
random-translation fr --strategy targeted --roots "/usr/share/locale:/opt/myapp/share/locale"
```

## Using with Phosphor and XScreenSaver

Phosphor is a retro terminal-style screensaver that can display text produced by a shell
command. Different builds and front ends expose the command field under different names:

- On macOS (e.g. `brew install xscreensaver`) the field is usually labeled **Shell Cmd**
  under Display Text in the Phosphor settings dialog.
- On Linux and many XScreenSaver front ends it is usually labeled **Program**.

### PATH note

XScreenSaver runs with a minimal environment and typically does not include `/usr/local/bin`
in `PATH`. Always use the **full path** to the script:

```
/usr/local/bin/random-translation --strategy cached de
```

### macOS sandbox note

On macOS, `/usr/bin/python3` is an Xcode Command Line Tools stub that invokes `xcrun`,
which is blocked inside the App Sandbox that XScreenSaver/Phosphor runs in. The script
automatically detects and skips this stub, preferring a real interpreter (e.g. from
Homebrew at `/opt/homebrew/bin/python3` or `/usr/local/bin/python3`). If no real Python
is found it falls back to a pure `od`/`awk` `.mo` reader that works without any external
tools. Build the cache in a terminal first (see above) so no filesystem scan is needed
at screensaver time.

### One-shot command

Paste directly into Phosphor's Shell Cmd or Program field:

```
/usr/local/bin/random-translation --strategy cached el,tr,es,hu
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

- **Build the cache first** with `--strategy deep` for the languages you care about.
- **Use `-n 1`** for single-line displays; use `-n N` for multi-line bursts.
- **Mix languages** on the command line to rotate phrases from multiple targets.
- **Use `--roots` + `--strategy targeted`** to include app-specific locale directories.
- **Adjust Phosphor `--delay` and `--scale`** to tune pacing and readability.
- **Log selections** if you want to review phrases later (wrap the command and append to a file).

## Troubleshooting

**`No .mo files found`**

- Build the cache: `random-translation --strategy deep --cache-mode write --refresh LANG`
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
  --strategy STR      Discovery strategy: quick|indexed|targeted|deep|cached
  --cache-mode MODE   Cache behavior: read|write|none
  --refresh           Force rebuild of the .mo list (applies when cache-mode is write or when using transient cache)
  --ttl SECS          Cache TTL in seconds (default 3600)
  --debug             Print debug information showing what each strategy and cache decision does
  -n N                Print N random msgstr strings only (useful for screensavers)
  --output MODE       Output mode: normal|json|yaml|filename  (default: normal)
  --cache-stats       Print cache age and entry count then exit
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
platform-specific assumptions without guards. Add new discovery strategies behind the
existing strategy dispatch and document them.

## License

ISC
