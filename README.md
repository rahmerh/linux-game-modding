# Linux modding scripts

A small set of scripts to make modding games on Linux easier while keeping the base game installation clean.

The core idea is:
- Mods are installed into separate folders
- All mods are merged into a single `merged/` directory
- The game is launched through an **overlayfs mount** that overlays `merged/` on top of the original game directory
- The base game install dir is never modified

These scripts are intentionally **folder-based** for simplicity.

These scripts are only for Steam games using Proton.

## How it works

Before launch:
1. All mod folders in `<game>/mods/` are merged into `<game>/merged/`
2. An overlayfs mount is created combining:
   - `lowerdir` → the game install directory
   - `upperdir` → the merged mods directory
3. The overlay is mounted at `<game>/run/`

The game is then launched from the overlay mount. When the game exits it will try to clean up by umounting the active dir. If cleanup fails or the script is killed, unmount it manually.

## Folder layout

Each game lives in its own folder:

```
./games/<GAME>/
  mods/
    001_FirstMod/
    002_SecondMod/
  merged/ # All merged mods, acts as the 'upperdir'
  work/ # Overlayfs work folder, do not edit manually.
  run/ # Overlayed mount folder, empty unless mount is active.
```

This folder structure will be created automatically when running the scripts.

## Usage

`launch-game <game-name>`

Starts the game, `<game-name>` can be the full name or an abbreviation.

## Supported games

- Cyberpunk 2077
    - Abbreviations: `cbp`, `cyberpunk`, `cyberpunk2077`
- Skyrim Special Edition
    - Abbreviations: `skyrim`, `es5`
