# Building Rusty-IDE

## Prerequisites

All platforms need:
- [Node.js](https://nodejs.org/) 20.19+ or 22.12+
- [Rust](https://www.rust-lang.org/tools/install) (stable toolchain)

### Platform-specific

| Platform | Requirement |
|----------|-------------|
| **macOS** | Xcode Command Line Tools (`xcode-select --install`) |
| **Windows** | [Microsoft C++ Build Tools](https://visualstudio.microsoft.com/visual-cpp-build-tools/) (MSVC v143 + Windows 10/11 SDK) |
| **Linux** | System libraries (see below) |

#### Linux system libraries

Debian/Ubuntu:

```bash
sudo apt install -y libwebkit2gtk-4.1-dev libappindicator3-dev librsvg2-dev patchelf libssl-dev
```

Fedora:

```bash
sudo dnf install -y webkit2gtk4.1-devel libappindicator-gtk3-devel librsvg2-devel openssl-devel
```

Arch Linux:

```bash
sudo pacman -S --needed webkit2gtk-4.1 libappindicator-gtk3 librsvg openssl
```

openSUSE:

```bash
sudo zypper install -y libwebkit2gtk-4_1-0-devel libappindicator3-devel librsvg-devel libopenssl-devel
```

## Install dependencies (first time only)

```bash
npm install
cd agent-sidecar && npm install && cd ..
```

## Build the application

The same command works on every platform:

```bash
npm run tauri build
```

This runs three steps automatically:

1. **Frontend** — `npm run build` (TypeScript + Vite)
2. **Sidecar** — `npm run build:sidecar` (bundles the Node.js sidecar into a single self-contained `server.js` via tsup, then copies it to `src-tauri/resources/sidecar/`)
3. **Tauri** — compiles the Rust app in release mode and bundles the app + Node.js runtime + sidecar

### Build a specific bundle target

```bash
# macOS — DMG only
npm run tauri build -- --bundles dmg

# Linux — AppImage only
npm run tauri build -- --bundles appimage

# Linux — deb only
npm run tauri build -- --bundles deb

# Linux — rpm only
npm run tauri build -- --bundles rpm

# Windows — NSIS installer only
npm run tauri build -- --bundles nsis

# Windows — MSI only
npm run tauri build -- --bundles msi
```

## Output

### macOS

| Bundle | Path |
|--------|------|
| App bundle | `src-tauri/target/release/bundle/macos/Rusty-IDE.app` |
| DMG | `src-tauri/target/release/bundle/dmg/Rusty-IDE_0.1.0_<arch>.dmg` |

### Windows

| Bundle | Path |
|--------|------|
| NSIS installer | `src-tauri/target/release/bundle/nsis/Rusty-IDE_0.1.0_<arch>-setup.exe` |
| MSI | `src-tauri/target/release/bundle/msi/Rusty-IDE_0.1.0_<arch>.msi` |

### Linux

| Bundle | Path |
|--------|------|
| AppImage | `src-tauri/target/release/bundle/appimage/Rusty-IDE_0.1.0_<arch>.AppImage` |
| Debian | `src-tauri/target/release/bundle/deb/Rusty-IDE_0.1.0_<arch>.deb` |
| RPM | `src-tauri/target/release/bundle/rpm/Rusty-IDE_0.1.0-1.<arch>.rpm` |

> **AppImage** is the most portable single-file option across distributions. **deb** targets Debian/Ubuntu/Mint, **rpm** targets Fedora/RHEL/openSUSE.

## Bundled Node.js runtime

The bundled Node binary lives in `src-tauri/bin/rusty-node-<target-triple>` and is excluded from git (downloaded locally). You need the matching binary for each target platform:

| Target triple | Platform | Binary name |
|---------------|----------|-------------|
| `aarch64-apple-darwin` | macOS Apple Silicon | `rusty-node-aarch64-apple-darwin` |
| `x86_64-apple-darwin` | macOS Intel | `rusty-node-x86_64-apple-darwin` |
| `x86_64-pc-windows-msvc` | Windows 64-bit | `rusty-node-x86_64-pc-windows-msvc.exe` |
| `x86_64-unknown-linux-gnu` | Linux 64-bit (most distros) | `rusty-node-x86_64-unknown-linux-gnu` |
| `aarch64-unknown-linux-gnu` | Linux ARM64 | `rusty-node-aarch64-unknown-linux-gnu` |

### Downloading Node binaries

Download from <https://nodejs.org/dist/> (use Node v22.x, e.g. v22.23.1) and extract the `node` binary:

**macOS:**

```bash
# Apple Silicon Mac (M1/M2/M3/M4)
cd src-tauri/bin
curl -LO https://nodejs.org/dist/v22.23.1/node-v22.23.1-darwin-arm64.tar.gz
tar xzf node-v22.23.1-darwin-arm64.tar.gz
cp node-v22.23.1-darwin-arm64/bin/node rusty-node-aarch64-apple-darwin
chmod +x rusty-node-aarch64-apple-darwin
rm -rf node-v22.23.1-darwin-arm64*

# Intel Mac
cd src-tauri/bin
curl -LO https://nodejs.org/dist/v22.23.1/node-v22.23.1-darwin-x64.tar.gz
tar xzf node-v22.23.1-darwin-x64.tar.gz
cp node-v22.23.1-darwin-x64/bin/node rusty-node-x86_64-apple-darwin
chmod +x rusty-node-x86_64-apple-darwin
rm -rf node-v22.23.1-darwin-x64*
```

**Windows (run in PowerShell on a Windows machine):**

```powershell
cd src-tauri\bin
curl.exe -LO https://nodejs.org/dist/v22.23.1/node-v22.23.1-win-x64.zip
tar xf node-v22.23.1-win-x64.zip
copy node-v22.23.1-win-x64\node.exe rusty-node-x86_64-pc-windows-msvc.exe
Remove-Item -Recurse node-v22.23.1-win-x64*
```

**Linux:**

```bash
# x86_64
cd src-tauri/bin
curl -LO https://nodejs.org/dist/v22.23.1/node-v22.23.1-linux-x64.tar.xz
tar xf node-v22.23.1-linux-x64.tar.xz
cp node-v22.23.1-linux-x64/bin/node rusty-node-x86_64-unknown-linux-gnu
chmod +x rusty-node-x86_64-unknown-linux-gnu
rm -rf node-v22.23.1-linux-x64*

# ARM64
curl -LO https://nodejs.org/dist/v22.23.1/node-v22.23.1-linux-arm64.tar.xz
tar xf node-v22.23.1-linux-arm64.tar.xz
cp node-v22.23.1-linux-arm64/bin/node rusty-node-aarch64-unknown-linux-gnu
chmod +x rusty-node-aarch64-unknown-linux-gnu
rm -rf node-v22.23.1-linux-arm64*
```

> The binary **must** be present in `src-tauri/bin/` with the exact target-triple name before running `npm run tauri build`, otherwise the build fails with "missing sidecar binary".

## Cross-platform builds via CI

You cannot natively build Windows or Linux binaries from macOS. The recommended approach is **GitHub Actions** using a matrix of OS runners. Each runner builds natively for its own platform:

```yaml
# .github/workflows/build.yml
name: Build
on:
  workflow_dispatch:

jobs:
  build:
    strategy:
      fail-fast: false
      matrix:
        include:
          - os: macos-14      # Apple Silicon
            target: aarch64-apple-darwin
          - os: macos-13      # Intel
            target: x86_64-apple-darwin
          - os: ubuntu-22.04
            target: x86_64-unknown-linux-gnu
          - os: windows-latest
            target: x86_64-pc-windows-msvc

    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-node@v4
        with:
          node-version: 20

      - uses: dtolnay/rust-toolchain@stable
        with:
          targets: ${{ matrix.target }}

      - name: Install Linux deps
        if: runner.os == 'Linux'
        run: |
          sudo apt update
          sudo apt install -y libwebkit2gtk-4.1-dev libappindicator3-dev librsvg2-dev patchelf libssl-dev

      - name: Install deps
        run: |
          npm install
          cd agent-sidecar && npm install && cd ..

      - name: Download Node sidecar binary
        working-directory: src-tauri/bin
        run: |
          if [ "${{ runner.os }}" = "Windows" ]; then
            curl -LO https://nodejs.org/dist/v20.20.2/node-v20.20.2-win-x64.zip
            tar xf node-v20.20.2-win-x64.zip
            cp node-v20.20.2-win-x64/node.exe "rusty-node-${{ matrix.target }}.exe"
          else
            OS="linux"; EXT="tar.xz"; ARCH="x64"
            if [ "${{ runner.os }}" = "macOS" ]; then OS="darwin"; EXT="tar.gz"; fi
            if [ "${{ matrix.target }}" = "aarch64-apple-darwin" ]; then ARCH="arm64"; fi
            if [ "${{ matrix.target }}" = "x86_64-apple-darwin" ]; then ARCH="x64"; fi
            curl -LO "https://nodejs.org/dist/v20.20.2/node-v20.20.2-${OS}-${ARCH}.${EXT}"
            tar xf "node-v20.20.2-${OS}-${ARCH}.${EXT}"
            cp "node-v20.20.2-${OS}-${ARCH}/bin/node" "rusty-node-${{ matrix.target }}"
            chmod +x "rusty-node-${{ matrix.target }}"
          fi
          rm -rf node-v20.20.2-*
        shell: bash

      - name: Build
        run: npm run tauri build

      - uses: actions/upload-artifact@v4
        with:
          name: Rusty-IDE-${{ matrix.target }}
          path: |
            src-tauri/target/release/bundle/**/*
```

## Development

For development with hot-reload, run the sidecar and frontend separately:

```bash
# Terminal 1 — sidecar (hot-reload via ts-node)
cd agent-sidecar && npm run dev

# Terminal 2 — Tauri dev (frontend + Rust with HMR)
npm run tauri dev
```

> In dev mode the bundled sidecar is **not** auto-spawned; you run it manually for hot-reload.
