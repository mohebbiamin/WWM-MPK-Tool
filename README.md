# Where Winds Meet — MPK Asset Toolkit

**Extract, inspect and rebuild `.mpk` archives of Where Winds Meet** (Everstone Studio / NetEase) — including the current patched game files that older tools can no longer open.

One portable Windows executable. Graphical interface **and** command line. Nothing to install.

![version](https://img.shields.io/badge/version-1.0.0-blue)
![platform](https://img.shields.io/badge/platform-Windows%2010%2F11%20x64-informational)
[![version](https://img.shields.io/github/v/release/mohebbiamin/WWM-MPK-Tool?sort=semver)](https://github.com/mohebbiamin/WWM-MPK-Tool/releases)

> The game changed how its archives are indexed and started scrambling file paths on purpose, so old MPK extractors now output garbage or fail outright. This tool reads the current format, recovers the real file names and folder structure, and can pack everything back into a working archive for modding and localization work.

---

## Preview
<img width="1919" height="1019" alt="image" src="https://github.com/user-attachments/assets/a76e2a69-1a25-43b0-a7c5-408743e56123" />
<img width="1919" height="1019" alt="image" src="https://github.com/user-attachments/assets/84f5f8ce-11b0-4768-8a75-ba1442b756f5" />

---

## Contents

- [What it does](#what-it-does)
- [Results on a real archive](#results-on-a-real-archive)
- [Download](#download)
- [Using the window (GUI)](#using-the-window-gui)
- [Using the command line](#using-the-command-line)
- [Modding: replace or add files](#modding-replace-or-add-files)
- [About the `.lua` files](#about-the-lua-files)
- [About audio and music](#about-audio-and-music)
- [FAQ](#faq)
- [Release model](#release-model)
- [Files in this repository](#files-in-this-repository)
- [Changelog](#changelog)
- [Author & credits](#author--credits)

---

## What it does

- **Opens current-gen archives** — both base-game files and post-update patches (`LT*.mpk`, `Patch*.mpk`, `Resources.mpk`).
- **Real file names, real folders** — reconstructs the original directory tree instead of dumping numbered `.bin` files.
- **Correct extensions** — identifies content by looking inside each file (audio, banks, textures, fonts, videos, archives…), so you get `.lua`, `.wem`, `.bnk`, `.png`, `.dds`, `.ttf`, `.ogg` and so on instead of unknown blobs.
- **Audio scanner** — walks the whole archive and reports which audio streams are actually inside, and how many.
- **Integrity checker** — confirms in one click that the files it extracted are structurally valid, so you know nothing was corrupted.
- **Rebuilder** — packs an edited folder back into a working `.mpk` + index pair, including files you added.
- **Patch database viewer** — reads `Patch.mpkdb` and cross-references it with an archive index, so you can see which assets a specific update touched.
- **Reports** — writes a machine-readable manifest plus a simple one-line-per-file text list you can open in Excel.
- **Works without the archive too** — if you only have the index file, it opens in a read-only mode and tells you exactly what is missing.

---

## Results on a real archive

Measured on `LT11.mpk` from the current version (13.4 MB, 3,568 indexed files):

```text
extracted              3,586 files (3,568 indexed + 18 leftover blocks) in ~13 seconds
byte-for-byte verify   3,586 compared, 0 differences
rebuild round trip     rebuilt archive is identical to the original, byte for byte
file names recovered   3,209 fully confirmed (90%), the rest with visible gaps
integrity check        3,568 / 3,568 files structurally valid
audio found            0 - correct, this archive holds scripts only
```

No data is lost and nothing is guessed: whatever the tool writes back is checked against the original before packing.

---

## Download

Get the pre-compiled executable from the **[Releases](https://github.com/mohebbiamin/WWM-MPK-Tool/releases)** page.

- **File:** `wwm-mpk-tool.exe` (~12.9 MB, single file)
- **Requires:** Windows 10 or 11, 64-bit. No Python, no runtime, no installer.

> Windows SmartScreen may warn about an unsigned download — the file simply isn't code-signed. Check the file size and version in its properties, and run it in a sandbox first if you prefer.

---

## Using the window (GUI)

Double-click **`wwm-mpk-tool.exe`**. Three ways to load an archive:

1. **Drag & drop** a `.mpk` (or `.mpkinfo`) file straight onto the exe icon.
2. Type or paste the path into the **MPK** box and press **Enter**.
3. Click **Open…** (or press **Ctrl+O**).

The matching `.mpkinfo` index must be in the **same folder** as the `.mpk`.

| Button | What it does |
|---|---|
| **Extract ALL** | unpacks everything, keeping the game's folder structure |
| **Extract selection** | unpacks only the highlighted rows (`Ctrl` / `Shift` for multi-select) |
| **Rebuild archive** | packs the output folder back into a working archive |
| **Scan for audio** | reports every audio stream actually present in the file |
| **Check Lua integrity** | confirms the script files were extracted correctly |
| **Open Patch.mpkdb** | loads the patch database and cross-references it with the open archive |
| **About / Info** | version, author, contact links and technical documentation |

The table shows each file's name, type, detected extension, size, location and whether its name was fully confirmed. Use the **Filter** box to narrow by name, content type or category, and tick **show un-indexed blocks** to also list leftover data the game no longer references.

![About Info Window](screenshots/about.png)

---

## Using the command line

```bash
wwm-mpk-tool.exe gui                        # open the window (default when run with no arguments)
wwm-mpk-tool.exe about                      # version, author, contact links

wwm-mpk-tool.exe info     LT11.mpk          # summary of an archive
wwm-mpk-tool.exe list     LT11.mpk          # list files with recovered names
wwm-mpk-tool.exe list     LT11.mpk --filter ui

wwm-mpk-tool.exe extract  LT11.mpk -o out --orphans --verbose
wwm-mpk-tool.exe extract  Resources.mpk -o audio_out --filter wem
wwm-mpk-tool.exe verify   LT11.mpk out      # compare extracted files with the archive
wwm-mpk-tool.exe rebuild  LT11.mpk out -n MODDED.mpk

wwm-mpk-tool.exe scanaudio Resources.mpk    # audio inventory of a file
wwm-mpk-tool.exe checklua  LT11.mpk         # structural validity of the scripts
wwm-mpk-tool.exe db Patch.mpkdb --join Resources.mpkinfo
```

Handy options: `--orphans` (also recover leftover blocks), `--filter` (by name, type or extension), `--name-mode id` (name files by ID instead of path), `--limit`, `--json`, `--verbose`.

---

## Modding: replace or add files

```bash
# 1. unpack - keep the folder, the manifest inside it is needed later
wwm-mpk-tool.exe extract LT11.mpk -o mymod --orphans

# 2. edit files in place, replace them, or drop new ones into the folder
#    (do not rename or move the existing ones)

# 3. repack
wwm-mpk-tool.exe rebuild LT11.mpk mymod -n MODDED.mpk
```

The rebuild step regenerates the index automatically, reports how many files changed, and creates a matching index file next to the new archive. New files added to the folder are packed as extra entries.

---

## About the `.lua` files

They are **compiled game scripts**, not plain text — that is normal and expected.

- They will look like nonsense in Notepad. That is true of every compiled script in this game, not a problem with the extraction.
- They are **not encrypted and not compressed**. Nothing was lost while unpacking: the tool verifies each file and can reproduce the original archive byte for byte.
- To read or edit the logic you need a Lua 5.4 decompiler. Be aware that this game's compiler writes a slightly non-standard header, so some off-the-shelf decompilers complain before they even start.
- If you just want to know whether extraction went well, run **Check Lua integrity** — it tells you how many files are structurally valid.

---

## About audio and music

Music and voice are **not** inside the script archives (`LT*.mpk`). They live in the asset containers — `Resources.mpk` and the asset patch archives.

Use **Scan for audio** on a file to see immediately whether it contains audio, and how many streams. On a script archive it will honestly report *no audio data here* instead of inventing hits.

---

## FAQ

**The tool says the container is missing.**  
You opened an index without its archive. Put the matching `.mpk` next to the `.mpkinfo` and press **Reload** — or continue in read-only mode to just browse the file list.

**Some names contain `~`.**  
The game itself damages parts of its stored paths. Every unrecoverable character is marked with `~` instead of being silently guessed or dropped.

**What is an "un-indexed block"?**  
Leftover data still physically inside the archive but no longer listed by the index — usually an older version of a file. Extract them with **show un-indexed blocks** / `--orphans`.

**Does it upload or touch anything online?**  
No. It is offline, reads files you point it at, and writes only where you tell it to.

**Will it break my game files?**  
It never modifies the archive you open; rebuilding always writes new files. Still keep a backup copy of anything you plan to replace in the game folder.

---

## Release model

This is a **binary-only release**. The executable is the product: it is self-contained, needs no runtime and works on a clean Windows 10/11 x64 install.

The internal format research behind it — how the archives are indexed, how names are stored and repaired — is deliberately **not documented here**. That knowledge is the point of the project, and publishing it would only make it easy to rehost the tool under someone else's name.

- Bug reports, feature requests and "can you support archive X" → **Telegram: [@Amin_o0x](https://t.me/Amin_o0x)** (fastest) or open an issue here.
- Source is not published. If you have a legitimate need for it (integration, audit, learning), ask on Telegram first rather than opening a request here.
- The tool is free to use, redistribute **as the unmodified executable**, and credit where it helps.

---

## Files in this repository

```text
wwm-mpk-tool.exe            the program - run this
README.md                   this documentation
screenshots/                interface preview images
```

---

## Changelog

### v1.0
- Supports the current (post-update) archive and index format
- Real file and folder names recovered, with per-file confidence marking
- Content detection by inspecting each file, so extensions are correct
- Audio scanner that reports only genuinely present streams
- Integrity checker for extracted scripts
- Rebuilder with automatic index regeneration and change reporting
- Patch-database viewer with cross-referencing against any archive index
- Read-only mode when only an index file is available
- Drag & drop, Enter-to-load, Ctrl+O
- About window with version and contact links; the same information is embedded in the exe's file properties
- Byte-exact round trip verified (extract → rebuild produces a file identical to the original)
- Application icon and Windows file properties

---

## Author & credits

Developed by **Amin Mohebbi** (`mohebbi`).

- **GitHub:** [mohebbiamin](https://github.com/mohebbiamin) — repository: [WWM-MPK-Tool](https://github.com/mohebbiamin/WWM-MPK-Tool)
- **ResHax profile:** [mohebbi on ResHax](https://reshax.com/profile/11716-mohebbi/)
- **Telegram:** [@Amin_o0x](https://t.me/Amin_o0x)

**Disclaimer.** For research, modding and localization purposes only. The tool ships no game content, works fully offline and bypasses no online protection. All game trademarks, art and audio belong to NetEase and Everstone Studio.
