---
name: lazarus
description: Use when reverse engineering shipped or dead software.
version: 1.0.0
author: PALIMPSEST
license: MIT
platforms:
- linux
- windows
- macos
metadata:
  hermes:
    tags:
    - reverse-engineering
    - binary-analysis
    - interoperability
    category: engineering
---

# LAZARUS: reverse engineering for live and dead software

## When to Use
- A shipped, undocumented or discontinued app must be understood before it is driven,
  replaced or rebranded.
- You need a capability map of a binary: what protocols, formats and scripting surfaces
  it exposes.
- A vendor has disappeared and their installers, drivers or project files need preserving
  and decoding.
- Skip this for ordinary source-repository architecture review — read the source instead.

**Use when** analysing shipped, undocumented or discontinued software: PE/ELF/Mach-O
binaries, Qt/QML apps, Electron apps, Go/Rust/.NET/Android targets, firmware, custom
binary project formats, scripting surfaces, or reviving an app whose vendor is gone.

Derived from `leanrea` (aicodeking, MIT). Extended for our reality: **Windows-first
targets, Qt/QML apps (not Electron), custom binary document formats, and discontinued
software whose vendor no longer exists.** You already know the tools; this file is
routing, first-moves, and traps.

## Non-negotiables
- **Preserve before probing.** Hash every original (SHA-256) in an evidence ledger before
  you extract, patch, or run anything. Work on copies, never on the user's install.
- **Mark every claim**: `seen` (file:offset, symbol, string), `inferred` (reasoned from
  evidence), `unknown` (you did not find it). Not-found is *unknown*, never *absent* — a
  bounded search proves nothing universal.
- **Ask before running, attaching, or modifying.** Static reading is not observed execution.
  Launching the target, attaching a debugger, or editing a binary each need scoped approval.
- Static analysis is not a licence. Interoperability does not grant redistribution rights.
- Turns cost more than bytes: batch probes into one call, cap output, aim for 2-4 calls.

## Route first
`file -b T` and magic bytes decide everything. Cheapest inventory, then slice, then
(only if needed) disassemble. Never `cat` a large or minified file —
`rg -o '.{0,200}NEEDLE.{0,200}' F`. Search extracted trees with `rg -uu` (bundlers hide
code in `.webpack/`, `.vite/`).

## First moves

**Windows PE (our primary target class)**
```sh
T=./app.exe; D=/tmp/re/$(basename $T); mkdir -p $D
file $T; objdump -h $T | head -14
echo '--- DLLS'; objdump -p $T | awk '/DLL Name:/{print $3}' | sort -u
echo '--- EXPORTS'; objdump -p $T | awk '/\[Ordinal\/Name Pointer\]/,/^$/' | head -60
echo '--- STRINGS'; strings -a -n 6 $T | rg -v '^[_$.]' | head -400
objdump -d -M intel $T > $D/dis.txt
```
Then slice: `rg -n 'NEEDLE' $D/dis.txt | head`, walking back to the nearest label above
the hit to find the enclosing function.

**Linux ELF**
```sh
file $T; readelf -hlSW $T | head -40; readelf -dW $T | rg NEEDED
nm -D --defined-only $T | head -100; objdump -d -M intel $T > $D/dis.txt
strings -a -n 6 $T | head -400
```

**Electron**: `npx -y @electron/asar extract app.asar X`; entry = `jq .main X/package.json`;
surface = `rg -uu -o '.{0,120}(ipcMain\.(handle|on)|ipcRenderer\.(invoke|send)|contextBridge\.exposeIn\w*World|preload:).{0,160}' X`.

**Go**: `go version -m T` usually answers toolchain, module and deps alone.
**Rust**: `cargo audit bin T`, or strings for `/rustc/<sha>` and `<crate>-<ver>/src`.
**Android**: `jadx -d X --show-bad-code a.apk`, then `rg`.
**SQLite**: copy the db **plus `-wal`**; `sqlite3 'file:C?mode=ro' '.schema'`.
**Firmware**: `binwalk -e F` / `unblob -e X F`, then route each part back through this table.

## Qt / QML applications — leanrea has no path for this, ours must
A large share of pro AV and creative tooling is Qt, not Electron. Signature: imports of
`Qt6Core.dll`, `Qt6Qml.dll`, `Qt6Quick.dll`, and `qrc:/` strings.
```sh
rg -o '[A-Za-z0-9_]+\.qml' T | sort -u        # the whole UI map, from strings alone
rg '^qrc:/' T | sed 's|/[^/]*$||' | sort | uniq -c | sort -rn
rg -o '[A-Za-z0-9_]+\.(png|svg|ico|ttf)' T | sort -u
```
QML is usually compiled into the binary or a `.rcc` blob, which makes **the UI a
rebranding surface**: dialog titles, page names, colour tokens, icon asset names.
Extract with `rcc --list` if a `.rcc` exists, else carve the qrc blob and parse the tree.
Qt also stores settings via `QSettings` — check `HKCU\Software\<vendor>\<app>` on Windows.

## Custom binary document formats
When a `.xyz` project file starts with a readable magic (`xxd -l 256 F`) and entropy sits
below ~7 bits/byte, assume a **tagged record tree**, not compression. Hypothesise
`[type:u8][len:u32][payload]`, payload often `[hash:u32][sub_len:u32][bytes]`.
Validate by walking records, asserting every length stays in bounds, and requiring the
walk to end exactly at EOF. Then dump to JSON and diff files written by different
versions — that difference *is* the schema history.

## Scripting and binding surfaces — check before rebuilding anything
Search exports for a scripting bridge before writing a replacement:
```sh
rg -o 'CSharp_\w+|Python_\w+|SWIG_\w+|lua_\w+|JS_\w+' <exports-file>
rg -i 'script|console|macro|automation|plugin|python' T | head -40
```
A SWIG/C# surface (`CSharp_<Namespace>_<Class>_<Method>`) means the app is **already
programmable**, and driving it beats reimplementing it. Look for `*ScriptingDialog*`,
`*Console*`, `*Plugin*` in strings and QML names.

## Reviving discontinued software — the LAZARUS sequence
1. **Archive first, always.** Vendor S3/CDN links die silently. Pull installers, runtimes
   (VC++ redist), drivers and sample files NOW; hash them; record source and date. Verify
   `curl -sIL` returns HTTP 200 and note `Content-Length` before trusting a link.
2. **Read the sunset notice for safety recalls.** A discontinued device can carry an
   unresolved fire, burn, laser or electrical advisory. Find it before plugging anything in.
3. **Map cloud dependencies.** Which features need a dead server — login, catalog, sync,
   remote access? Confirm the offline subset actually runs offline.
4. **Find the scripting/plugin surface** (above): the cheapest interoperability win.
5. **Identify the real protocol.** Look for NDI/Spout/Syphon, HTTP, WebSocket and raw
   sockets in imports (`WS2_32`, `WINHTTP`, `WININET`). The wire protocol outlives the UI.
6. **Then decide**: drive it, replace it, or reimplement one verified path.

## Speed patterns
- `nm -u` / `objdump -p` imports are the capability map; strings are the intent map.
- To find who uses a string: locate it in `.rdata`, find the cross-reference, read the
  enclosing function — do not read every instruction.
- Diff inventories (exports, imports, module list, `go version -m`) before diffing code.
- A C++ mangled name in a call target already tells you the protocol.
- Rebuilding: implement the minimum, then test it against the original on identical inputs.

## Traps
- `rg` skips hidden dirs by default — use `-uu` on extracted trees.
- zsh expands a bare `==x`; quote echo markers.
- Headless SSH sessions do not attach to the interactive desktop: screen/display
  enumeration there is unreliable, not evidence of a missing display.
- A missing tool is not a blocker: use the fallback column, or ask before installing.

## Install (only on request)
`apt install binutils file sqlite3` · `brew install jadx apktool radare2 binwalk wabt` ·
`npm i -g @electron/asar` · `dotnet tool install -g ilspycmd` · `cargo install cargo-audit`.
