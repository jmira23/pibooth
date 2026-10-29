# Running pibooth locally (macOS / non-Raspberry Pi)

## Requirements

- Python 3.10–3.12 (`brew install python@3.12`). Do not use 3.14: `pygame` has no wheel for it and the source build fails with `ImportError: cannot import name 'Font'`. Create the venv with `/opt/homebrew/bin/python3.12 -m venv .venv`.
- A webcam (used through OpenCV)

## Install

From the repository root:

```bash
# Create an isolated Python environment in the .venv folder
python3 -m venv .venv

# Activate it so `python` and `pip` use this environment (repeat in each new terminal)
source .venv/bin/activate

# Install pibooth and its dependencies in editable mode (code changes apply without reinstalling)
pip install -e .

# Install OpenCV, used by pibooth to read from your webcam
pip install opencv-python
```

## Run

```bash
# Start the photobooth with verbose (debug) logging
pibooth -v
```

Other options:

```bash
# Use a custom config folder instead of ~/.config/pibooth
pibooth ./my_config

# Open the configuration menu
pibooth --config

# Restore the default configuration
pibooth --reset

# Disable writing logs to a file
pibooth --nolog

# Show only warnings and errors
pibooth -q
```

Config is stored in `~/.config/pibooth/` by default.

## Notes

- `RPi.GPIO` is only installed on ARM boards, so hardware buttons and LEDs are unavailable. Use the keyboard (capture: `P`, print: `Ctrl+E`, fullscreen: `Ctrl+F`, restart/reload: `R`). Keys are hardcoded: to use the spacebar for capture, change `pygame.K_p` to `pygame.K_SPACE` in `find_capture_event()` in `pibooth/booth.py` and restart.
- Without a Pi camera or DSLR, pibooth falls back to the OpenCV camera.

## Optional: use a DSLR instead of the webcam

```bash
# Install the native gphoto2 library required by the DSLR Python binding
brew install libgphoto2

# Add DSLR camera support (gphoto2 Python package)
pip install -e ".[dslr]"
```

Then:

1. Connect the DSLR via USB, switch it on and set it to manual/photo mode. Check that it is on the [gphoto2 supported list](http://gphoto.org/proj/libgphoto2/support.php).
2. Close any app that may grab the camera (macOS often starts `ptpcamera` automatically). Free it with:
   ```bash
   # Kill the macOS process that claims the camera so gphoto2 can access it
   killall ptpcamera
   ```
3. Run `pibooth -v`. The camera is auto-detected: a connected DSLR is used first, and if a webcam is also present they are combined (DSLR for the final capture, webcam for the preview). Without a DSLR it falls back to the webcam.

Run `pibooth-diag` to check which cameras are detected.

## Optional: use a printer

```bash
# Add printing support (pycups, requires CUPS, which is built into macOS)
pip install -e ".[printer]"
```

Then:

1. Add your printer in macOS System Settings > Printers & Scanners (or at http://localhost:631), so it is registered in CUPS.
2. List the CUPS printer names:
   ```bash
   # Show the printer names known to CUPS
   lpstat -p
   ```
3. Open `pibooth --config` and, in the `PRINTER` section, set:
   - `printer_name`: the CUPS name from the previous step (`default` uses the system default printer).
   - `auto_print`: number of copies printed automatically (0 = ask the user).
   - `max_pages`: pages printed before a paper/ink warning (-1 = unlimited).
4. Run `pibooth -v`. A print button appears after a capture (keyboard: `Ctrl+E`).

Tip: `pibooth-printcfg` helps set up the printer paper size and options.

## Tests

```bash
# Install the test runner
pip install pytest

# Run the test suite in the tests folder
pytest tests
```
