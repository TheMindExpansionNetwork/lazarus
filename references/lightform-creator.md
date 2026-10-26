# LAZARUS reference: Lightform Creator 2.4.1

Worked example from a real engagement. Target: a discontinued Qt6 projection-mapping
application. Claims are `seen` in the artifacts unless marked inferred.

## Target inventory (seen)
`Lightform.exe` PE32+ x86-64, 6 sections, 19,335,768 B.
Companion DLLs: `LightformEngine.dll` (44.9 MB), `LightformRenderer.dll` (3.3 MB),
`LightformCommon.dll`, `LightformCreatorLibrary.dll` (SWIG bridge).
Bundled assets: `Lightform-Built_In-1_0_0.zip` … `-4_0_0.zip` (~435 MB total).

## Imports that revealed the architecture (seen)
| Import | Meaning |
|---|---|
| `Qt6Core/Gui/Qml/Quick/QuickWidgets` | UI is QML, not Electron |
| `Processing.NDI.Lib.x64.dll` | **NDI output is a first-class feature** |
| `WINHTTP.dll`, `WININET.dll`, `WS2_32.dll` | device discovery + network protocol |
| `LightformCreatorLibrary.dll` | the scripting bridge |

## The scripting bridge (seen)
Exports shaped `CSharp_Lightform_<Class>_<Member>`:
`CSharp_Lightform_Application___getActiveProject`, `___getAssetBrowser`,
`___getExecutableFilePath`, `___getScriptSelfTestState`.
SWIG registration symbols (`SWIGRegisterExceptionCallbacks_LightformCreatorLibrary`)
confirm a SWIG-generated C# API. `ScriptingDialog.qml` exists (seen), so a script UI is
shipped (inferred). **Consequence:** the app is already programmable; driving it is
cheaper than replacing it.

## QML surface (seen, selected)
`DesignPage`, `ControlPage`, `EffectEditor`, `ScanDialog`, `SelectDeviceDialog`,
`CameraStreamWindow`, `ScriptingDialog`, `EnableLabsDialog`, `ImageShareServersDataModel`,
`NetworkAccessPointsDataModel`, `GPUMemoryProfilerWindow`, `LoginPage`,
`SelectArchiveProjectDirectoryPage`.
Rebranding surface = dialog titles, page names, icon asset names, colour tokens.

## The `.lit` project format (seen; parser hypothesis)
Entropy 5.94 bits/byte, 27% printable — mixed binary, not compressed. Contains the ASCII
`Lightform Binary Document` preceded by a `litb` tag.
Observed header walk:
```
04 | 08 00 00 00 | c1 63 eb 07 6c 69 74 62      -> type=4 len=8, payload ends 'litb'
04 | 21 00 00 00 | e9 2f 07 f6 19 00 00 00
   'Lightform Binary Document'                  -> type=4 len=33 (4 hash + 4 sublen + 25)
```
Working hypothesis: `[type:u8][len:u32][hash:u32][sub_len:u32][payload]`.
Validate by walking to exact EOF before trusting it.

Schema tokens recovered from strings (seen): `Project`, `Canvas`, `Slide 1`,
`Master Slide`, `Asset`, `Effect`, `Property Array`, `PRERENDERED`, `PROCEED`,
`COLOR_RGB`, `ROTATE_0`. A recurring `%N!D7` prefix is likely a shared object/id tag.

## Safety finding (seen, vendor doc)
LF2/LF2+ firmware `0.10.6`, `2.1.19`, `2.1.22` can leave the cooling fan off — fire and
burn risk. Safe: `2.1.23` or `2.4.1.0`+. Always check a discontinued device for an
unresolved safety recall before powering it.

## Lessons that generalise
1. Check for a scripting bridge before planning a rebuild.
2. NDI/Spout in imports means the video path is already networkable — use it.
3. Qt/QML apps are rebrandable at the resource layer.
4. A vendor's own sunset notice is the best safety document you will get.
