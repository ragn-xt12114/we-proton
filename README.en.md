# WE-Proton

[中文](README.md) | English

## About

On Linux (Wine / Proton), the WeGame client fails to download games, places windows in the wrong spot, misbehaves on input and shows black screens. This project is based on GE-Proton and DW-Proton: it starts from GE-Proton, adds DW-Proton's patches (compatibility fixes for anime games and others), and fixes the Wine / Proton / vkd3d-proton bugs behind those problems. The goal is to make the WeGame client itself, and games without kernel-level anti-cheat (single-player games, for example), run properly.

Companion launcher: [wegame-launcher](https://github.com/SHORiN-KiWATA/wegame-launcher) installs and runs WeGame in one step.

This fork additionally documents and verifies AK accelerator's mode 5 (wintun2socks) data plane; see [AK accelerator (mode 5)](#ak-accelerator-mode-5).

## About AI

The patches added by this project were written and debugged with the help of AI. The symptoms, root cause and verification of each patch are documented in [`we/fixes/`](we/fixes/) (in Chinese).

## Competitive games and anti-cheat

- This project does not fix or support competitive games such as Delta Force or VALORANT. Issues about them will be closed.
- This project does not adapt to ACE or any other anti-cheat, and does not accept requests or code that bypass anti-cheat or relate to cheating.
- Linux is not a platform supported by ACE. Even if a game with ACE runs, the account may still be banned for an abnormal environment. You bear the consequences, including bans, of using this project.

## Demo

https://github.com/user-attachments/assets/42438b93-5a36-4557-8a0f-4b65b53d5e07

## AK accelerator (mode 5)

> Status: verified with `WE-Proton ge11-11-31` and `shorin-proton-wrapper`'s private network namespace (2026-11-13).

AK's mode 5 uses wintun2socks: it creates an `ak_wintunocks` TUN inside the prefix's private netns and relays game traffic through AK nodes. Requirements:

1. Install `passt` (for example `sudo apt install passt` or `sudo pacman -S passt`).
2. Enable the prefix's private network with `shorin-proton-wrapper --netns on` (skip if already enabled; this is a configuration step, not a per-run flag).
3. Start Steam and the AK client (`AK.exe` from AKPlatform) in the same wrapper session; both must share the same netns.
4. If you use an AK-side Wine compatibility solution, apply its patches only after AK's self-check finishes (the log shows `客户端文件检测完成`); the self-check restores files under `core/`.
5. Select mode 5 in AK, accelerate, then launch the game through Steam (for example `steam -applaunch 3527290`).

Verification: PEAK (AppID 3527290) reached the online world with pure Wine mode 5, node 8166(8295);
no disconnect for about 7 minutes. TUN captured 21,824 UDP packets; the node side showed 21,807 A5 5A frames (approximately 1:1).

Known limitations:

- Under Wine, AK first tries mode 4, fails, then automatically falls back to mode 5; this is expected.
- Wine's built-in wintun read path and `GetOwnerModuleFromUdpEntry` do not fully match AK's expectations. This repository only provides the Wine/Proton runtime; the AK-side Wine compatibility work is provided by the AK-side solution and is not included here.
- The verified game data plane used Photon UDP (5058/5055/5056); the TCP relay path was not exercised in this run and needs separate verification for TCP/WSS scenarios.
- An accelerator only changes the network path and does not change this project's position on competitive games or anti-cheat; accelerator nodes, accounts and game compliance are the user's responsibility.

## Installation

1. Download `we-proton-<version>.tar.xz` from [Releases](https://github.com/SHORiN-KiWATA/we-proton/releases)
2. Extract it with `tar` into your launcher's Proton directory, e.g. for Steam:

   ```
   mkdir -p ~/.local/share/Steam/compatibilitytools.d
   tar -xf we-proton-<version>.tar.xz -C ~/.local/share/Steam/compatibilitytools.d/
   ```

   Lutris and Heroic also pick up Proton builds from Steam's directory, so the command above works for them as well; you can also point `-C` at their own Proton directory.

   > Do not extract with 7-Zip. Many DLLs in the package are symlinks, and recent 7-Zip versions skip them as "dangerous" and leave empty files behind, which breaks the prefix in confusing ways. If you saw `Dangerous link path was ignored` while extracting, delete the extracted directory, extract again with the command above, and recreate any prefix that was created with it.

3. Select `we-proton-<version>` in the launcher. wegame-launcher uses the newest WE-Proton it finds by default; to pick one by hand, use Settings → Runner in the GUI, or run

   ```
   wegame-launcher runner we-proton-<version>
   ```

## Building from source

Needs docker. The first full build takes a few hours:

```
git clone https://github.com/SHORiN-KiWATA/we-proton.git
cd we-proton
we/build.sh --name we-proton-<version>             # output in build/
we/build.sh --name we-proton-<version> --install   # also install to "~/.local/share/proton/runners/WE-Proton Latest"
```

The script fetches the submodules, patches them with `patches/protonprep-valve-staging.sh` (GE-Proton's patches, then DW-Proton's and WE-Proton's) and runs `make redist` in the Steam Linux Runtime SDK container. The details, the test programs, the diagnostic tools and how to follow new upstream releases are described in [`we/README.md`](we/README.md) (in Chinese).

## Repository layout

- `patches/wine/`, `patches/vkd3d-proton/we-*.patch`: WE-Proton's patches
- `patches/dwproton/`: Wine patches DW-Proton has and GE-Proton doesn't
- The rest of `patches/` and `patches/protonprep-valve-staging.sh`: GE-Proton's patches and patching script (`protonfixes/0003`–`0006` come from DW-Proton; `protonfixes/0051` is WE-Proton's own). The submodules stay at the upstream commits; the script applies all patches before the build
- `we/fixes/`: one report per patch (symptoms, root cause, fix, verification)
- `we/tests/`: test programs and their output before and after each fix
- `we/diag/`: diagnostic patches and scripts, not part of the release build
- `we/build.sh`: the build script
- `we/sync/`, `we/upstream-sync.md`: scripts and a step-by-step guide for following new GE-Proton and DW-Proton releases
- Everything else is GE-Proton's / Proton's own content

## Source and licenses

- Proton's top-level contents: BSD-3-Clause (Valve Corporation, see [`LICENSE`](LICENSE) and [`LICENSE.proton`](LICENSE.proton))
- Wine: Valve's Wine plus the patches in `patches/`, LGPL-2.1
- vkd3d-proton: upstream vkd3d-proton plus the patches in `patches/vkd3d-proton/`, LGPL-2.1; the dxil-spirv subproject and the patch to it are MIT
- Other components: see the `LICENSE` / `COPYING` files in their directories, and `LICENSE`, `LICENSE.OFL` and `PATENTS.AV1` in the release package
- The scripts and documents under `we/` are under BSD-3-Clause as well

## Acknowledgements

- [Proton](https://github.com/ValveSoftware/Proton) by Valve and CodeWeavers
- [GE-Proton](https://github.com/GloriousEggroll/proton-ge-custom) by GloriousEggroll
- [DW-Proton](https://dawn.wine/dawn-winery/dwproton) by Dawn Winery
- [Wine](https://www.winehq.org/), [DXVK](https://github.com/doitsujin/dxvk), [vkd3d-proton](https://github.com/HansKristian-Work/vkd3d-proton) and the other projects Proton is built from
