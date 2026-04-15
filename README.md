# setup-swift

A GitHub Action that installs a specific Swift version on **macOS**, **Linux**, and **Windows**.

If Swift is already present at the requested version (e.g. inside a Swift Docker container), installation is skipped automatically.

## Usage

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: DeveloperBeau/setup-swift@v1
    with:
      swift-version: 'latest'  # optional, defaults to 'latest'
  - run: swift build
```

### Version input

`swift-version` accepts:

- `latest` — resolves to the newest release on swift.org
- Partial versions (`6`, `6.3`) — resolves to the highest matching release
- Exact versions (`6.3.0`) — used as-is (no API call)

### Matrix build

```yaml
jobs:
  build:
    strategy:
      fail-fast: false
      matrix:
        include:
          - runner: macos-latest
          - runner: ubuntu-latest
          - runner: windows-latest
    runs-on: ${{ matrix.runner }}
    steps:
      - uses: actions/checkout@v4
      - uses: DeveloperBeau/setup-swift@v1
      - run: swift build -v
      - run: swift test -v
```

### Inside a Swift container

The action detects the existing Swift version and skips installation if it matches:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    container:
      image: swift:6.3
    steps:
      - uses: actions/checkout@v4
      - uses: DeveloperBeau/setup-swift@v1  # detects Swift 6.3, skips install
      - run: swift build
```

### Reading the resolved version

```yaml
- id: swift
  uses: DeveloperBeau/setup-swift@v1
  with:
    swift-version: 'latest'
- run: echo "Installed Swift ${{ steps.swift.outputs.version }}"
```

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `swift-version` | `latest`, partial (`6`, `6.3`), or exact (`6.3.0`) | `latest` |
| `skip-verify-signature` | Skip GPG (Linux) / pkg (macOS) signature verification | `false` |

## Outputs

| Output | Description |
|--------|-------------|
| `version` | Fully-resolved Swift version that was installed |

## How it works

| Platform | Method | Signature verification |
|----------|--------|------------------------|
| macOS | Downloads `.pkg` from swift.org, installs toolchain | `pkgutil --check-signature` (Apple signer) |
| Linux | Downloads tarball from swift.org, extracts to `/opt/swift` | GPG verify against `swift.org/keys/all-keys.asc` |
| Windows | Installs via `winget` (Visual Studio components + Swift toolchain) | Handled by winget |

The action verifies the installed version matches the requested version after installation.

## CI matrix

This repository tests the action against the following combinations on every push and PR:

| Platform | Runner | Swift versions tested |
|----------|--------|-----------------------|
| macOS | `macos-latest` | `latest`, `6.3`, `5` |
| Linux | `ubuntu-latest` | `latest`, `6.3`, `6` |
| Windows | `windows-latest` | `latest`, `6.3`, `6` |

### Why no Swift 5 on Linux or Windows?

- **Linux**: swift.org does not publish Swift 5 tarballs for Ubuntu 24.04 (the current `ubuntu-latest`). The latest Swift 5 release only ships binaries for older distros (Ubuntu 18.04/20.04/22.04), so installation on `ubuntu-latest` would 404.
- **Windows**: Swift 5 was distributed as a standalone installer, not through `winget`. The `Swift.Toolchain` winget package used by this action only provides Swift 6.x. Installing Swift 5 on Windows requires a different code path that is out of scope for this action.

Swift 5 remains supported on macOS because swift.org continues to ship Xcode toolchain `.pkg` installers for Swift 5 that are compatible with current macOS runners.

## License

MIT
