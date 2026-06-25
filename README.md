# Linux modding scripts

A small set of scripts to make modding games on Linux easier while keeping the base game installation clean.

The core idea is:
- Mods are installed into separate folders
- All mods are merged into a single `merged/` directory
- The game is launched through an **overlayfs mount** that overlays `merged/` on top of the original game directory
- Any game generated files end up in `sink/`
- The base game install dir is never modified

These scripts are intentionally **folder-based** for simplicity.

These scripts are only for Steam games using Proton.

## How it works

An overlayfs mount is created with this layer order, from bottom to top:

- game install directory
- merged mods directory at `<game>/merged/`
- persistent game-write directory at `<game>/sink/`

In overlayfs terms:

- `lowerdir` → `<game>/merged/:<game install dir>`
- `upperdir` → `<game>/sink/`
- `workdir` → located at `<game>/work/`, should not be edited manually.

The overlay is mounted at `<game>/run/`, from which the game is launched. 
Any files created or changed by the game are written to `<game>/sink/`.
This directory is persistent, to be safe any generated game files should be moved into a mod dir (for example `999-sink` so they're always copied last.
When the game exits it will try to clean up by umounting the active dir. 
If cleanup fails or the script is killed, unmount it manually.

## Folder layout

Each game lives in its own folder:

```
./games/<GAME>/
  mods/
    001_FirstMod/
    002_SecondMod/
  merged/ # All merged mods, rebuilt when the mod list changes.
  sink/ # Persistent overlayfs upperdir for files created or changed by the game.
  work/ # Overlayfs work folder, do not edit manually.
  run/ # Overlayed mount folder, empty unless mount is active.
```

This folder structure will be created automatically when running the scripts.

## Usage

`bootstrap-game <game-name>`

Sets up any required files before you can start installing mods, if required.

`merge-mods <game-name>`

Merges all mods into the merged dir, you should run this after every change to your mod list.

`launch-game <game-name>`

Starts the game, `<game-name>` can be the full name or an abbreviation.

## Supported games

- Cyberpunk 2077
    - Abbreviations: `cbp`, `cyberpunk`, `cyberpunk2077`
- Skyrim Special Edition (1.6.1170)
    - Abbreviations: `skyrim`, `es5`
- Minecraft
    - Abbreviations: `minecraft`, `mc`
