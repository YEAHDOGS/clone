# Clone — ephemeral in-browser Linux (prototype)

A single static page. Pick a small Linux distro, it boots entirely in the
browser via v86 (x86 emulation in WebAssembly). RAM-only: no persistence,
nothing written to disk. Close the tab, the machine is gone.

## Layout

- `index.html` — the whole app (no frameworks, no build step)
- `vendor/` — v86 vendored locally (`libv86.js`, `v86.wasm`)
- `bios/` — SeaBIOS + VGA BIOS (from the v86 project)
- `isos/` — local ISO copies used for dev/test only; the page itself
  points at public mirrors (see CORS note below)

## Serve it

Any static server works:

```sh
cd ~/workspace/clone && python3 -m http.server 8123
# open http://localhost:8123/
```

Query params (dev/test):

- `?local=1` — fetch ISOs from `./isos/` instead of public mirrors
- `?autoboot=tinycore|alpine|slax` — boot one immediately on load

## Distros (all 32-bit — v86 is an i686 emulator, x86_64 will not boot; all < 300 MB, none hosted by this page)

| Distro | Size | Public mirror URL |
|---|---|---|
| DSL 4.4.10 | 50 MB | https://archive.org/download/damn-small-linux-4.4.10/dsl-4.4.10.iso |
| Tiny Core (current) | 25 MB | https://archive.org/download/tiny-core-current_202412/TinyCore-current.iso |
| Alpine 3.22.1 (x86) | 222 MB | https://dl-cdn.alpinelinux.org/alpine/v3.22/releases/x86/alpine-standard-3.22.1-x86.iso |
| Slax 7.0.8 (i486) | 226 MB | https://archive.org/download/20260924_20260924_1127/slax-English-US-7.0.8-i486.iso |

DSL 4.4.10 is the one verified to reach a working graphical desktop in
headless testing. TinyCore-current's 6.x kernel panics under v86
("Attempted to kill the idle task"; with `acpi=off noapic` it hangs at
"Booting the kernel"). Alpine/Slax are wired but not boot-tested.

## Known limits (by design, not bugs)

- **CORS**: archive.org's file servers do NOT send
  `Access-Control-Allow-Origin` (verified 2026-10-08), so browsers block
  cross-origin ISO fetches. The page tries the public URL, shows
  "mirror blocked", and falls back to a same-origin `./isos/` copy.
  For production the ISOs need a CORS-enabled host — or same-origin
  hosting, which is also better for privacy (no third-party mirror
  learns which distro was booted).
- **No guest networking** in v86. The guest cannot reach the internet.
- **Speed**: emulation is roughly an order of magnitude slower than
  native; full desktops (KDE/GNOME) are out of reach — hence small ISOs.
- **v86 API**: current builds export `V86`, not `V86Starter` (renamed
  upstream). This page uses `V86`.
- **Chromium 152** blocks `http://127.0.0.1` navigation (Local Network
  Access checks, no working flag). Tested via `file://` +
  `--allow-file-access-from-files` instead.
