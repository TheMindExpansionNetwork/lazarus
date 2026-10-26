# LAZARUS

**Reverse engineering for live and dead software.**

A skill for AI agents that reverse-engineer shipped, undocumented, or discontinued
software using plain local CLI tools — no MCP server, no tool registry, no setup.

```
git clone https://github.com/TheMindExpansionNetwork/lazarus
cp -R lazarus ~/.hermes/profiles/<profile>/skills/     # or your agent's skills dir
```

If your agent has no skills support, paste `SKILL.md` into its rules or system prompt.
It is plain text.

## Why this exists

[leanrea](https://github.com/aicodeking/leanrea) proved that a short routing note beats a
large MCP tool surface: same accuracy, far less time, cost and context. LAZARUS starts from
that thesis and closes the gaps we hit on a real engagement — reviving a **discontinued
Windows Qt6 application** whose vendor no longer exists.

leanrea is excellent and assumes macOS. Our targets are not macOS.

| Gap | What LAZARUS adds |
|---|---|
| macOS-first (`otool`, `lipo`, `codesign`) | **Windows PE as a primary target class** — full `objdump` first-move |
| **No Qt/QML path** | Qt detection; QML as a rebranding and analysis surface |
| No custom binary formats | Tagged-record-tree hypothesis, validation method, schema diffing |
| No binding surfaces | SWIG / C# / Python / Lua export discovery, "check before rebuilding" |
| No revival workflow | The 6-step LAZARUS sequence for dead vendors |
| Silent on safety | Archive-first; check for unresolved recalls **before** powering hardware |

## What is in the skill

**Non-negotiables** — preserve and hash before probing; mark every claim `seen` /
`inferred` / `unknown`; ask before running, attaching or modifying; static analysis is not
a licence.

**First moves** — one paste-and-run command each for Windows PE, Linux ELF and Electron, each
returning the whole inventory in a single tool call.

**Router** — Qt/QML, Electron/asar, minified bundles, Hermes/bytenode, Mach-O, ELF/PE,
Go buildinfo, Rust cargo-auditable, .NET, Android APK, Flutter snapshot hash, WASM, SQLite,
plists, pcap/HAR, firmware, websites, live runtimes.

**Custom binary document formats** — how to tell a tagged record tree from compression, how to
validate the walk ends exactly at EOF, and how diffing two versions' dumps recovers schema
history.

**Scripting and binding surfaces** — a SWIG/C# surface means the app is already programmable,
and driving it beats reimplementing it.

**The LAZARUS sequence** — for software whose vendor is gone: archive installers now, read the
sunset notice for safety recalls, map dead cloud dependencies, find the scripting surface,
identify the real protocol, then decide: drive it, replace it, or reimplement one verified path.

## Worked example

`references/lightform-creator.md` walks a real target end to end: a discontinued Qt6
projection-mapping application. Imports revealed native NDI and a SWIG C# bridge; strings
recovered the full QML UI map from the binary alone; and a custom `.lit` project format was
characterised down to its record header hypothesis.

Read it if you want to see the method applied rather than described.

## Credit

Built on the ideas and structure of [leanrea](https://github.com/aicodeking/leanrea) by
aicodeking, which in turn credits the [REA](https://github.com/morluto/rea) team.
Not affiliated with either.

MIT licensed.
