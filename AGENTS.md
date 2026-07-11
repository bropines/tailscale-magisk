# Magisk Tailscaled: Developer Agent Guidelines

This document contains mandatory rules, architectural principles, and instructions for any AI coding assistant (agent) working with the **Magisk Tailscaled** repository.

---

## 🏗️ Core Architecture & Tech Stack

This repository packages Tailscale for Android devices rooted with Magisk or KernelSU.

1. **Core (Go Daemon + CLI)**: A single binary `tailscale.combined` compiled with `ts_include_cli` build tag. Sourced from `tailscale/tailscale`.
   - Running it as `tailscale` triggers the CLI interface.
   - Running it as `tailscaled` triggers the background daemon.
2. **System-wide VPN Routing**: The daemon creates a `tailscale0` TUN interface and uses system policy routing (`ip route`, `ip rule`, `iptables`) to route `100.64.0.0/10` traffic.
3. **Control Socket**: Standardized Unix socket path is `/data/adb/tailscale/tailscaled.sock`.
4. **Service Management**: Done via `tailscaled.service` (delegated from `/system/bin/tailscaled.service`), allowing `start`, `stop`, `restart`, and `status`.

---

## 📂 Project Layout

* [`tailscale/`](file:///home/pinus/projects/tailscaled/tailscale/) — Contains configuration files and service management scripts.
  * [`settings.ini`](file:///home/pinus/projects/tailscaled/tailscale/settings.ini) — Core path and parameter settings.
  * [`scripts/`](file:///home/pinus/projects/tailscaled/tailscale/scripts/) — Daemon startup and interface routing control scripts.
* [`system/bin/`](file:///home/pinus/projects/tailscaled/system/bin/) — Wrapper scripts placed into `/system/bin` at boot time for easy command line access.
* [`patches/`](file:///home/pinus/projects/tailscaled/patches/) — Atomic `.patch` files applied sequentially to pristine Tailscale source code during automated builds.
* [`.github/workflows/build.yml`](file:///home/pinus/projects/tailscaled/.github/workflows/build.yml) — Automated compiler and release workflow.

---

## 🛠️ Build System & Automation

Binaries are compiled and packaged automatically using GitHub Actions:

### 1. Scheduler
The builder runs automatically every 3 days via cron (`0 0 */3 * *`). It checks the latest stable release tag from `tailscale/tailscale`. If it's newer than the latest tag in our repository, a build is triggered.

### 2. Multi-Architecture Compilation
Compilation is performed using Android NDK toolchain compilers targeting 4 architectures:
* **arm64** (`GOARCH=arm64`, `aarch64-linux-android21-clang`)
* **arm** (`GOARCH=arm GOARM=7`, `armv7a-linux-androideabi21-clang`)
* **x86** (`GOARCH=386`, `i686-linux-android21-clang`)
* **x86_64** (`GOARCH=amd64`, `x86_64-linux-android21-clang`)

### 3. Binary Compression & Packaging
All compiled binaries are compressed using UPX (`--lzma --best`), placed under `files/<arch>/tailscale.combined`, and packed into the Magisk module release ZIP file.

---

## 🩹 Patching Guidelines

All modifications to the Go code of Tailscale must be managed as atomic `.patch` files under the [`patches/`](file:///home/pinus/projects/tailscaled/patches/) folder:
- Do not create a single monolithic patch.
- Keep patches focused, prefixing them with sequential numbers (e.g. `0001-...patch`).
- Ensure they apply cleanly using `patch -p1` on the checked-out Tailscale version.
