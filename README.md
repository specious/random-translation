# Random Translation Picker

**A small, portable tool to pick a random installed GNU gettext `.mo` file for one or more languages and emit translations.**

This tool is ideal for testing, demos, language learning, and screensavers. It is portable, fast when cached, and works on macOS, Linux, and other Unix-like systems.

## Features

- **Discovery strategies:** `quick`, `indexed`, `targeted`, `deep`, `cached`
- **Cache control:** `read`, `write`, `none` with TTL and `--refresh`
- **Output formats:** `normal` (PO-like), `json`, `yaml`, `filename`
- **Sampling:** print `-n N` random translated strings (great for screensavers)
- **Locale matching:** `es`, `es_ES`, `es-ES`, `es_ES.UTF-8`
- **Fallbacks:** uses `python3` or `msgunfmt` when available; graceful degradation otherwise
- **Debugging:** `--debug` shows discovery and cache decisions

## Install

Install the script to a standard location so it is available system-wide:

```bash
# Preferred: install sets owner and mode in one step
sudo install -m 755 random-translation /usr/local/bin/

# Or copy manually
sudo cp random-translation /usr/local/bin/
sudo chown root:root /usr/local/bin/random-translation
sudo chmod 755 /usr/local/bin/random-translation
```

Adjust the prefix if you prefer a different install location.

## Build a persistent cache

Building a cache once makes subsequent runs fast and reliable. Replace the language list with the languages you study.

```bash
# Recommended: uses system indexers when available
random-translation --strategy indexed --cache-mode write --refresh el tr es hu

# Fallback: exhaustive (slower) scan if indexed misses files
random-translation --strategy deep --cache-mode write --refresh el tr es hu
```

After building the cache you can run without `--refresh` for quick sampling.

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

Show the path that would be read:

```bash
random-translation es --dry-run
```

Target a specific set of roots:

```bash
random-translation fr --strategy targeted --roots "/usr/share/locale:/opt/myapp/share/locale"
```

## Using with Phosphor and XScreensaver

Phosphor is a retro terminal style screensaver that can display text produced by a shell command. Different builds and front ends expose the command field under different names:

- On macOS builds the UI often labels the field **Display Text > Shell Cmd**. Paste a one-line command there.
- On Linux and many XScreensaver front ends the field is usually labeled **Program**. Enter the program invocation there.

Two common ways to feed `random-translation` into Phosphor:

**One-shot command**
Phosphor runs the command and displays its output:

```
/usr/local/bin/random-translation --strategy cached el,tr,es,hu
```

Enter that exact command into Phosphor’s Shell Cmd or Program field.

**Pipe mode**
If your Phosphor supports pipe mode, run Phosphor with `--pipe` and `--program` so it reads from a long-running producer:

```
phosphor --scale 4 --delay 80000 --pipe --program 'random-translation -n 1 el tr es hu'
```

**Recommended Phosphor options**

- `--scale 2` or `--scale 4` to adjust pixel size
- `--delay 80000` to slow the character reveal for readability
- `--pipe` when feeding a continuous stream

## Tips for screensaver use and language learning

- **Build the cache first** for the languages you care about so the screensaver is responsive.
- **Use `-n 1`** for single-line displays; use `-n N` for multi-line bursts.
- **Mix languages** on the command line to rotate phrases from multiple targets.
- **Use `--roots` + `--strategy targeted`** to include app-specific locale directories.
- **Adjust Phosphor `--delay` and `--scale`** to tune pacing and readability.
- **Log selections** if you want to review phrases later (wrap the command and append to a file).

## Troubleshooting

- If you see `No .mo files found`:
  - Build the cache: `--strategy indexed --cache-mode write --refresh LANG` or `--strategy deep --cache-mode write --refresh LANG`
  - Use `--roots` with `--strategy targeted` to point at known locale directories
- If JSON/YAML output fails, ensure `python3` or `msgunfmt` is installed
- Use `--debug` to inspect discovery and cache decisions

## Contributing

- Fork, add tests or improvements, and open a pull request
- Keep changes portable and avoid platform-specific assumptions without guards
- Add new discovery strategies behind feature flags and document them

## License

ISC
