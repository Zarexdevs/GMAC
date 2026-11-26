# GMAC

**GTA VI on Apple Silicon Macs.** GMAC runs the Xbox Series build of Grand Theft Auto VI on an M-series Mac, from a
copy of the game you own, through Wine, Apple's D3DMetal and Rosetta 2.

> [!IMPORTANT]
> **GMAC is not playable yet.** No game files are included here, now or ever. You need your own copy of the game.
> No code is published yet either; this page tracks progress and the plan.

## Where it stands (26 November 2026)

Tested on a **MacBook with an M5 and 16 GB of memory**.

| Stage | Status |
| --- | --- |
| Boot, first-run screens, intro video with sound | works, 45 to 60 fps |
| Rockstar logo, intro art, legal and settings screens | works, rendered correctly |
| Story-mode load ("loading screen" art) | works, a steady 60 fps (Series S mode), 25 to 35 fps as a Series X |
| Story intro (`FBK_STRY000_INTRO`) | streams in, and its dialogue plays |
| Story mode | **the 3D world draws:** the city, its lighting and characters on screen, dialogue and HUD, at 1 to 4 fps; stays up 6+ minutes |
| Picture quality | ghosting and silhouetted characters: about 200 draws a frame are still dropped (mesh shaders Metal cannot build yet) |
| Gameplay | not yet |

### Latest findings

- **The world draws (26 November).** The black main view was the wrong console display plane: on Series S the
  finished scene is on plane 0 (1080p) and the HUD on plane 1 (4K). GMAC now shows plane 0 with plane 1 blended over
  it, so loading screens, the HUD and the city all show.
- **No more GPU hangs.** Mesh shaders Metal cannot build (over its 32 KB mesh-output limit, or ones Apple's shader
  compiler crashes on) are refused and learned automatically, instead of hanging the GPU.
- **Memory was the frame rate.** In the world the game used only about 1.6 of 10 CPU cores: the Mac was swapping
  to disk, because the GPU held copies of nearly all the game's memory plus several GB of waste. Overlapping copies
  are now merged (they held 3.6 GB for 2.7 GB of memory), copies nothing uses are freed, copy windows are 256 KB
  instead of 4 MB, and idle textures are given up after 30 s. GPU memory now holds at about 10 GB instead of
  hitting 13.3 GB within three minutes, swapping nearly stopped, and the world went from 0.3 to 1 fps to 1 to 4 fps.
- **Rosetta-safe write tracking.** GMAC rewrites about 98,000 of the game's 256-bit AVX stores as 8-byte moves when
  it loads the game, so tracking which pages the game writes no longer freezes Rosetta. A faster copy path replaces
  Wine's kernel copy for game memory.
- **Next:** fix the dropped mesh-shader draws (the ghosting), then let CPU and GPU work overlap: today VIctory waits
  for the GPU after each of about 28 submissions a frame.

## Target

1. The opening title card and cutscene on screen.
2. The 3D world drawing at **2 to 4 fps**.
3. In the long run, **50 fps** in the world.
4. A setup a Mac owner can run in a few commands.

## How it works

The game's Xbox code is x86-64, like a Windows program, so a loader maps it and stands in for the console:

- **Rosetta 2** runs the x86-64 code.
- **Wine** (CrossOver's, built from source) provides Windows.
- **D3DMetal**, from Apple's Game Porting Toolkit, turns Direct3D 12 into Metal.
- The console's GPU shaders are recompiled to DXIL, **ahead of time and natively on the Mac**.

## Built on VIctory and five-star-recomp

GMAC now runs the game on [VIctory](https://github.com/Marvisak/VIctory) through
[VIctory-macOS](https://github.com/pragyaangaur/VIctory-macOS) (both GPL-3.0), with CrossOver Wine built from
CodeWeavers' source. It started on [five-star-recomp](https://github.com/ThatRecompDev/five-star-recomp) (GPL-3.0),
and the Mac work done there carries over. GMAC's changes will be published under the same licence.

## What GMAC adds for the Mac

- **Rendering on Metal:** pixel-shader inputs numbered as the console numbers them (the intro was one flat colour
  before), descriptor strides for D3DMetal, CPU-built rectangle lists, buffer views at GPU-only addresses, and shader
  hashes D3DMetal can cache by.
- **Fitting 16 GB:** merged, right-sized GPU copies of game memory that are freed when unused, idle textures given
  up, 1080p and lower render targets, half-size read-only textures, and a memory guard that stops the game before
  the Mac runs out.
- **Loading faster:** all 25,908 shaders compiled natively ahead of time instead of under Rosetta during the load,
  pipelines built ahead of time, buffered file streaming, and the game's AMD-only spin-wait instructions patched so they
  no longer fault through Rosetta's exception thread.
- **Getting through the load:** the game's GPU watchdog trap patched out, its 256-bit AVX stores made safe for
  Rosetta so write tracking can run, Series S mode for memory, the game's own upscaler instead of FSR 4, the right
  display plane, and 60 fps loading screens.
- **Staying stable:** giant and slow Metal shader compiles skipped instead of stalling the GPU, a clean quit that
  releases the GPU's memory (no restart needed between runs), a scan that cannot race the game's memory unmaps, and
  a log that names the shaders of a GPU submission that hangs.
- **Tooling:** a launcher with the memory guard, fps and load progress in the window title, and a fail-closed mute.

## Legal

GMAC contains no game code, assets or keys. Grand Theft Auto VI is a trademark of Take-Two Interactive and Rockstar
Games; this project is not affiliated with or endorsed by them.
