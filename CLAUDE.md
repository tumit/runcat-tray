# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Linux system tray app that displays an animated icon whose speed scales with CPU usage. Written in C using GTK3/libappindicator. Single-file implementation (`main.c`).

## Build

Requires: `libappindicator-gtk3`, CMake >= 3.10

```sh
mkdir -p build && cd build && cmake .. && make
make install  # installs icon assets to ~/.config/runcat/icons
```

AddressSanitizer build: `cmake -DCMAKE_BUILD_TYPE=ASAN ..`

## Run

```sh
./build/runcat          # default cat animation
runcat -l 6 -u 90       # set FPS lower/upper bounds
runcat -d /abs/path/to/icons/setname   # use custom icon set
```

## Architecture

**CPU sampling**: reads `/proc/stat` every 1s, calculates delta of (user+nice+system) jiffies, normalized by core count and 100 Hz kernel tick rate.

**Animation loop**: self-rescheduling GTK timeout callback. Frame delay is inversely proportional to CPU % between `FPS_L` (lower, default 6) and `FPS_H` (upper, default 90). Higher CPU = faster animation.

**Icon sets**: PNG/SVG frames in `~/.config/runcat/icons/{cat,dab,mona,partyblobcat}/`. Max 30 frames per set, sorted alphabetically by filename. Custom sets via `-d` flag.

**Tray menu**: AppIndicator with radio submenu for mode selection, CPU % label, and quit button. Modes are switchable at runtime.

**State**: All global variables. No dynamic allocation for frames (fixed-size arrays: 30 paths × 256 chars, path buffer 1024 chars).

## CLI Flags

- `-l` — lower FPS bound (animation speed at 0% CPU)
- `-u` — upper FPS bound (animation speed at 100% CPU)
- `-d` — absolute path to icon set directory
