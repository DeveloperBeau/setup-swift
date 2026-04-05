# setup-swift

A GitHub Action that installs a specific Swift version on **macOS**, **Linux**, and **Windows**.

If Swift is already present at the requested version (e.g. inside a Swift Docker container), installation is skipped automatically.

## Usage

```yaml
steps:
  - uses: actions/checkout@v4
  - uses: beauayres/setup-swift@v1
    with:
      swift-version: '6.3'  # optional, defaults to 6.3
  - run: swift build
```

### Matrix build

```yaml
jobs:
  build:
    strategy:
      fail-fast: false
      matrix:
        include:
          - runner: macos-latest
          - runner: ubuntu-24.04
          - runner: windows-latest
    runs-on: ${{ matrix.runner }}
    steps:
      - uses: actions/checkout@v4
      - uses: beauayres/setup-swift@v1
      - run: swift build -v
      - run: swift test -v
```

### Inside a Swift container

The action detects the existing Swift version and skips installation if it matches:

```yaml
jobs:
  build:
    runs-on: ubuntu-24.04
    container:
      image: swift:6.3
    steps:
      - uses: actions/checkout@v4
      - uses: beauayres/setup-swift@v1  # detects Swift 6.3, skips install
      - run: swift build
```

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `swift-version` | Swift version to install (e.g. `6.3`, `6.1`) | `6.3` |

## How it works

| Platform | Method |
|----------|--------|
| macOS | Downloads `.pkg` from swift.org, installs toolchain |
| Linux | Downloads tarball from swift.org, extracts to `/opt/swift` |
| Windows | Installs via `winget` (Visual Studio components + Swift toolchain) |

The action verifies the installed version matches the requested version after installation.

## License

MIT
