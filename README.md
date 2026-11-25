# GMAC

**GTA VI on Apple Silicon Macs.** GMAC runs the Xbox Series build of Grand Theft Auto VI on an M-series Mac, from a
copy of the game you own, through Wine, Apple's D3DMetal and Rosetta 2.

> [!IMPORTANT]
> **GMAC is not playable yet.** No game files are included here, now or ever. You need your own copy of the game.
> No code is published yet either; this page tracks progress and the plan.

## Where it stands (25 November 2026)

Tested on a **MacBook with an M5 and 16 GB of memory**.

| Stage | Status |
| --- | --- |
| Boot, first-run screens, intro video with sound | works, 15 to 30 fps |
| Rockstar logo, intro art, legal and settings screens | works, rendered correctly |
| Story-mode load ("loading screen" art) | works, 15 to 30 fps, about 6 to 9 minutes on 16 GB |
| Entering the game world | the world starts (its audio plays), the picture does not show yet |
| Gameplay | not yet |

## Target

1. The opening title card and cutscene on screen.
2. The 3D world drawing at **2 to 4 fps**.
3. In the long run, **50 fps** in the world.
4. A setup a Mac owner can run in a few commands.

## How it works

The game's Xbox code is x86-64, like a Windows program, so a loader maps it and stands in for the console:

- **Rosetta 2** runs the x86-64 code.
- **Wine** (Sikarugir build) provides Windows.
- **D3DMetal**, from Apple's Game Porting Toolkit, turns Direct3D 12 into Metal.
- The console's GPU shaders are recompiled to DXIL, **ahead of time and natively on the Mac**.

## Built on five-star-recomp

GMAC is built on [five-star-recomp](https://github.com/ThatRecompDev/five-star-recomp) (GPL-3.0), which loads the game
and translates its Xbox graphics and shaders for Windows PCs. That project has never been run on a Mac. GMAC is the
work that makes it run on Apple Silicon, and it will be published under the same licence. Ideas also came from
[VIctory](https://github.com/Marvisak/VIctory) and [VIctory-macOS](https://github.com/pragyaangaur/VIctory-macOS).

## What GMAC adds for the Mac

- **Rendering on Metal:** pixel-shader inputs numbered as the console numbers them (the intro was one flat colour
  before), descriptor strides for D3DMetal, CPU-built rectangle lists, buffer views at GPU-only addresses, and shader
  hashes D3DMetal can cache by.
- **Fitting 16 GB:** smaller GPU copies of game memory, 1080p and lower render targets, half-size read-only textures,
  and a memory guard that stops the game before the Mac runs out.
- **Loading faster:** all 25,908 shaders compiled natively ahead of time instead of under Rosetta during the load,
  pipelines built ahead of time, buffered file streaming, and the game's AMD-only spin-wait instructions patched so they
  no longer fault through Rosetta's exception thread.
- **Staying stable:** giant and slow Metal shader compiles skipped instead of stalling the GPU, a clean quit that
  releases the GPU's memory (no restart needed between runs), and a scan that cannot race the game's memory unmaps.
- **Tooling:** a launcher with the memory guard, fps and load progress in the window title, and a fail-closed mute.

## Legal

GMAC contains no game code, assets or keys. Grand Theft Auto VI is a trademark of Take-Two Interactive and Rockstar
Games; this project is not affiliated with or endorsed by them.
