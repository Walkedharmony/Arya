<p align="center">
  <img src="assets/branding/art3m1s-logo-v1.png" alt="Art3m1s icon" width="168">
</p>

# Art3m1s / Arya

A cross-platform visual novel runtime host written in Flutter. Flutter manages the game library, native window lifecycles, touch and pointer input, file sandboxing, audio/video playback, and platform integrations; the Rust core [`art3m1s-core`](https://github.com/Alphaly2K/art3m1s-core) handles script execution, game state evaluation, and offscreen layer composition.

Original creator: **[Alphaly2K](https://github.com/Alphaly2K)**  
License: **Mozilla Public License 2.0 (MPL-2.0)**

---

> [!IMPORTANT]
> The legacy iOS Flutter UI has entered **obsolete** status. `lib/shell/cupertino_shell.dart` and `lib/shell/liquid_glass_shell.dart` are retained solely for backward compatibility and rollback purposes. The modern iOS UI and runtime logic are maintained natively in [`native/ios/Art3m1sNative`](native/ios/Art3m1sNative).

---

## Features

- **Direct In-Place Import**: Import games from unpacked directories, single-volume or multi-volume PFS archives, and application sandboxes. Multiple PFS archives in the same folder are automatically organized into individual project entries without duplicating files.
- **Metadata & Cover Scraping**: Headless title detection on initial load with automated VNDB metadata and cover art fetching.
- **Project Isolation**: Unique project IDs isolate save data, game settings, and cover art, preventing cross-game save overwrites.
- **Adaptive Platform Shells**:
  - **Desktop**: Native window shell, automatically switching to a controller/keyboard-friendly PS5-style fullscreen TV interface when maximized.
  - **Android**: Switchable between modern **Material Design 3** and **MIUIX** design languages.
  - **iOS**: Native SwiftUI interface via `Art3m1sNative`.
- **High-Performance 60 FPS Runtime**: Real-time render loop forwarding pointer, keyboard, touch, right-click, hover, and drag events to the Rust runtime.
- **E-Mote 2D Interactive Animations**: Native playback of E-Mote character portraits supporting compressed **DXT5** and uncompressed **RGBA8 / LeRGBA8** textures at smooth 60 FPS with natural color rendering.
- **Software & Video Decoding**: Integrated static FFmpeg software decoding for opening movies, cutscenes, and transparent alpha-blended video effects (OGV/Theora).
- **Virtual Touchpad & Precise Cursor**: Optional relative touch trackpad for mobile devices with an arrow cursor for pixel-perfect clicking, hover triggers, and long-press dragging.
- **Comprehensive Audio Stack**: Independent BGM, sound effects (SE), and voice channels with audio focus and skip/seek synchronization.
- **Machine Translation & Patches**: Support for local JSON/JSONL translation patches as well as online translation APIs (OpenAI, Anthropic, DeepL, Google Translate, Baidu, and Youdao).
- **Developer & Debug Tools**: Real-time Profiler Overlay (FPS, frame time, texture allocations), in-game dockable HUD, session log viewer, log export, and about/license pages.

---

## Architecture

```text
Platform Shells
  Desktop Native + PS5 Big Screen / Material Shell / MIUIX Shell / Native iOS
        │
        ├─ LibraryActions
        │   ├─ In-app file browser / Android SAF / Documents picker
        │   ├─ Game title detection
        │   └─ VNDB metadata & cover queries
        │
        └─ PlayerScreen
            ├─ Input & lifecycle orchestration
            ├─ 60 FPS runtime loop
            ├─ MediaBridge (audio & video playback)
            └─ Shared surface texture / RGBA fallback
                     │
                  CoreBridge
            ┌────────┼────────┐
            │        │        │
       FileProvider  UI   Translation
       PFS/directory & host services
                     │ C FFI
                     ▼
                art3m1s-core
```

### CoreBridge

`lib/services/core_bridge.dart` is responsible for loading the target platform's native library (`libart3m1s_core`), holding runtime handles, and handling callbacks:

- Project and save file I/O (read, write, delete, query, copy);
- Media playback commands and video completion events;
- Dialog boxes, title changes, browser requests, and window status;
- System font discovery and text injection;
- Shared GPU texture submission and layer video texture updates.

Core calls are strictly serialized per runtime instance. Shutting down a runtime safely deregisters callbacks and stops active media tasks before freeing the native handle.

### FileProvider

`lib/services/file_provider.dart` provides a unified logical file namespace to the core:

- Single and multi-volume PFS archives (`.pfs`), and patch PFS files;
- Unpacked raw game directories;
- Isolated save states and global system configuration.

Resources are decoded using Shift_JIS or UTF-8 based on `system.ini`'s `CHARSET`. On Android, `MANAGE_EXTERNAL_STORAGE` grants direct file path resolution for seamless in-place playback without copying gigabytes of game data into app storage.

### MediaBridge

`lib/services/media_bridge.dart` coordinates audio and video playback:

- Background music (BGM), sound effects (SE), and character voices via host audio backends;
- Fullscreen movies and layer video effects decoded by `art3m1s-core`'s integrated FFmpeg runtime;
- Independent decoding timing ensures 24 FPS cutscenes do not bottleneck the 60 FPS gameplay engine.

---

## Platform Shells

| Platform | Shell Interface | Game Import Method |
|---|---|---|
| **macOS** | Windowed `macos_ui`; Fullscreen PS5 UI | Native file dialog / In-app TV browser |
| **Windows** | Windowed Fluent UI; Fullscreen PS5 UI | Native file dialog / In-app TV browser |
| **Linux** | Windowed Yaru/Material; Fullscreen PS5 UI | Native file dialog / In-app TV browser |
| **Android** | Modern Material Design 3 or MIUIX Shell | Native Android SAF (In-place import) |
| **iOS** | Native SwiftUI (`native/ios/Art3m1sNative`) | UIDocumentPicker / `Art3m1s/Games` |

---

## Building

### Prerequisites

- Flutter 3.47.x+
- Rust 2024 / 2021 Edition
- [vcpkg](https://github.com/microsoft/vcpkg) (for Darwin ANGLE and FFmpeg dependencies)
- Target platform SDKs (Android SDK/NDK, Xcode, Visual Studio, etc.)
- Precompiled `art3m1s-core` and `pfs-upk-rust` native libraries

### Unified Build CLI

The build runner automatically injects the current Git commit and `pubspec.yaml` version:

```bash
dart run tool/build.dart [all|ios|ios-obsolete|macos|android|windows|linux] \
  [--release|--profile|--debug] [--device-only] [--sign-only] [--krkr] [--siglus]
```

Example for building macOS release:

```bash
dart run tool/build.dart macos --release
```

Example for building Android release APK:

```bash
dart run tool/build.dart android --release
```

### Core Android Build (WSL Ubuntu)

To build the optimized `libart3m1s_core.so` for Android (`arm64-v8a`) using static FFmpeg:

```bash
bash art3m1s-core/scripts/build_core_android_wsl.sh
```

The resulting library is automatically installed to `art3m1s/android/app/src/main/jniLibs/arm64-v8a/libart3m1s_core.so`.

---

## Credits & License

Original author and creator: **[Alphaly2K](https://github.com/Alphaly2K)**

Licensed under the **[Mozilla Public License 2.0 (MPL-2.0)](LICENSE)**:
- File-level copyleft: Any modifications made to covered files in this repository must remain open source under MPL-2.0.
- Application packaging, custom extensions, and distribution binaries may be released under separate terms according to the Larger Work provisions of MPL-2.0.
