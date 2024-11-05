+++
title = 'GTA IV Installation on Linux Guide'
date = 2024-08-03T23:40:03+02:00
draft = true
+++
Setup guide for 1.0.8 Steam version (with Rockstar Games Launcher)

0. Install GTA IV from Steam
On the first run, it will also install RGL, which requires 1GB of free space.
Funny thing is, that it uses Windows fonts by default, so if you don't have them installed, in my case it uses Times New Roman each time I run the game.

1. Install DXVK (https://github.com/doitsujin/dxvk/releases)
The most important thing to make GTA IV running properly.
As the authors said: "A Vulkan-based translation layer for Direct3D 8/9/10/11 which allows running 3D applications on Linux using Wine".
So yeah, if you used it for better performance on Windows (like me), you will be shocked how performance outstand on Linux machine, as it should in dedicated environment.
Unpack d3d9.dll and dxgi.dll from x64 (or from x32 if you're running on 32-bits) into "GTAIV" directory inside installation root ([...]/steamapps/common/Grand Theft Auto IV)

2. Install ThirteenAG's FusionFix (https://github.com/ThirteenAG/GTAIV.EFLC.FusionFix)
Unpack entire archive content into "GTAIV" directory from installation root

3. Disable Steam Vulkan Shader Precaching
It is required for DXVK to make it work. Don't make the same mistake as mine, you'll wait for Steam to prerender Vulkan shaders only to find out that DXVK doesn't work.

4. Add WINEDLLOVERRIDES environment variable or (if it doesn't work) run winecfg on GTA IV's prefix with native d3d9, dxgi and dinput8 dlls
Crucial for Wine/Proton - that instruction indicates not to override these libraries.
n - native (use lib included with Windows or Application (GTA IV in this case))
b - bundled (use lib bundled with Wine/Proton installation)

If you want to use WINEDLLOVERRIDES:
In "Launch game options" provide: ```WINEDLLOVERRIDES="dinput8,dxgi,d3d9=n,b" %command```
If you want (like me) to override Wine/Proton config via winecfg:
```$ WINEPREFIX="<path_to_pfx_gta_iv_use>" winecfg```
In "Libraries" section add:
  dinput8 (native, bundled)
  dxgi (native, bundled)
  d3d9 (native, bundled)

5. Useful flags/running options:
  -norestriction
  -percentvidmem 100
  -availablevidmem 4096
  -nomemrestrict
  -norestrictions
  -noprecache
  -novblank

You can add them in commandline.txt file withing "GTAIV" directory or "Launch game options" inside GTA IV's "Game Properties" on Steam
