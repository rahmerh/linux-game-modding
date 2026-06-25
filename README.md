# Linux modding scripts

A small set of scripts to make modding games on Linux easier while keeping the base game installation clean.

The core idea is:
- Mods are installed into separate folders
- All mods are merged into a single `merged/` directory
- The game is launched through an **overlayfs mount** that overlays `merged/` on top of the original game directory
- Any game generated files end up in `sink/`
- The base game install dir is never modified

These scripts are intentionally **folder-based** for simplicity.

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
This directory is persistent. To be safe, any generated game files should be moved into a mod dir, for example `999-sink`, so they are always copied last.
When the game exits it will try to clean up by unmounting the active dir.
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

For Minecraft, this currently installs NeoForge only. The script will ask for the path to a NeoForge installer jar.

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
    - Abbreviations: `mc`

### Minecraft exceptions

Minecraft uses `$HOME/.minecraft` as the base install directory and launches through `minecraft-launcher`, not Proton.

The overlay mount is different from the Proton games:

- the launcher is run with `HOME` set to `games/Minecraft/run/`
- the overlay is mounted at `games/Minecraft/run/.minecraft`
- the original install remains `$HOME/.minecraft`

Because the launcher owns the running game process, cleanup only happens after the launcher and any game processes using the mount have exited.

Each Minecraft mod directory should contain files exactly where they should appear inside `.minecraft`.
`merge-mods` copies the contents of each folder in `games/Minecraft/mods/` into `games/Minecraft/merged/`.

For example:

```
games/Minecraft/mods/
  001_example_mod/
    mods/
      example-mod.jar
  002_example_shaderpack/
    shaderpacks/
      example-shader.zip
  003_example_resourcepack/
    resourcepacks/
      example-resource-pack.zip
```
