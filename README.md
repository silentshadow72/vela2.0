# Vela

A Minecraft Bedrock launcher for Android that runs unmodified OptiFine shaderpacks.

Bedrock cannot load Java shaderpacks. Vela imports your own Minecraft APK, runs it
in an isolated profile, preloads native mods into the game process, and replaces
the frame at `eglSwapBuffers` with output from its own shader runtime.

---

## Download

Go to **[Releases](../../releases)** and open the newest one. Under **Assets**,
download `VelaLauncher-<version>-debug.apk`.

- **arm64 only.** 32-bit devices cannot install it.
- **Android 9 or newer** (minSdk 28).
- Debug-signed, so it installs alongside the Play Store copy of Minecraft.

---

## What works

- Import your own Minecraft APK, one isolated profile per instance
- Four native mods bundled and installed automatically at launch, no importing:
  Custom Buffers, Dynamic Shadows, Vela Composite, Vela Shader Renderer
- Vela's own Content screen — worlds, resource packs, behaviour packs, skins,
  read straight off the instance's folders
- Dark amber UI, landscape, three tabs
- No telemetry: no Firebase analytics, no crash reporting, no Play Games

## What does not work yet

- **Shaders do not render.** The renderer loads and hooks `eglSwapBuffers`, but
  the shaderpack sources are desktop GLSL (`#version 150`) and Android's GLES
  rejects them. A translator exists and gets 12 of 27 programs through; the rest
  need explicit casts that GLSL ES requires and desktop GLSL does not.
- Nothing in the renderer has been verified on a GPU.
- Microsoft sign-in and Discord presence are stubbed out.
- World import/export is disabled (the native world-tools module is off).

---

## How it is built

Three Gradle modules — `app`, `minecraft`, `pojav_controls` — plus a native C++
renderer in `renderer/`.

| | |
| --- | --- |
| Kotlin | 2.3.0, Jetpack Compose |
| AGP | 8.13.2, Gradle 8.13 |
| SDK | compile 36, target 35, min 28 |
| JDK | 21 |
| NDK | 28.2.13676358 |

The APK is built by GitHub Actions on every push to `main`; the workflow is in
`.github/workflows/build.yml`. The repository holds the project as a single zip
which the workflow unpacks, builds the native renderer from, and assembles.

### Renderer

- Patches the first 16 bytes of `eglSwapBuffers` with `ldr x16,#8 / br x16`,
  keeping the displaced instructions in a trampoline. Refuses to hook when the
  prologue contains a PC-relative instruction rather than corrupting it.
- Executes a pack's `deferred → composite1..N → final` chain into its own
  render targets, per each pass's `DRAWBUFFERS` declaration.
- Adaptive quality: reads `/proc/meminfo`, picks a memory profile (2 GB gets four
  targets at half resolution, 9.6 MB instead of ~60 MB), and moves between four
  quality tiers targeting 25 ms frames.

---

## Credits

A fork of [LeviLaunchroid](https://github.com/LiteLDev/LeviLaunchroid) (Apache-2.0).

`CustomBuffers` and `DynamicShadows` are by their respective authors under MIT and
are redistributed unmodified. `libpreloader.so` is built from
[preloader-android](https://github.com/LiteLDev/preloader-android).

Not affiliated with Mojang or Microsoft. Minecraft is a trademark of Mojang AB.
