# ScanTailor Advanced for Mac

**ScanTailor Advanced for Mac** is an independent native macOS port of
[ScanTailor Advanced](https://github.com/4lex4/scantailor-advanced), an interactive
post-processing tool for scanned pages.

This fork targets **Apple Silicon (arm64) only**. It runs natively using Qt,
without Wine or Rosetta. It is not an official upstream macOS release.

<!-- Add a screenshot of the native Mac app once a window capture is available. -->

## What it does

Split scanned spreads into pages, deskew images, select content, adjust margins,
remove backgrounds, and produce cleaned-up pages ready for printing or assembly
into a document.

ScanTailor processes existing image files. It does **not** operate scanners,
install scanner drivers, perform OCR, or assemble multipage PDFs. Scan with
macOS Image Capture or your scanner's software first.

## Provenance and credits

The original ScanTailor Advanced project is maintained at
[4lex4/scantailor-advanced](https://github.com/4lex4/scantailor-advanced).
This port was forked from
[ScanTailor-Advanced/scantailor-advanced](https://github.com/ScanTailor-Advanced/scantailor-advanced),
which is itself a fork of that project. The initial Mac build used its
v1.2.1-era commit `5eaac18`.

The macOS work adds a native app bundle, bundled Qt libraries and plugins,
an app icon derived from upstream artwork, and Mac-specific display naming.
The image-processing features and artwork remain credited to the ScanTailor
and ScanTailor Advanced contributors.

See the [preserved upstream README](README-upstream.md) for feature descriptions,
history, and upstream Windows/Linux build notes. Those platform and release
instructions describe upstream, not releases from this Mac port.

## Build and run

Requirements:

- An Apple Silicon Mac.
- Xcode Command Line Tools (`xcode-select --install`).
- [Homebrew](https://brew.sh) installed at `/opt/homebrew`.

```bash
brew install cmake qt boost jpeg-turbo libpng libtiff
git clone https://github.com/ProbabilityEngineer/ScanTailor-Advanced-Mac.git
cd ScanTailor-Advanced-Mac

cmake -S . -B build-macos-arm64 \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_OSX_ARCHITECTURES=arm64 \
  -DCMAKE_PREFIX_PATH="$(brew --prefix qt);$(brew --prefix boost);$(brew --prefix jpeg-turbo);$(brew --prefix libpng);$(brew --prefix libtiff)" \
  -DBUILD_TESTS=OFF
cmake --build build-macos-arm64 --parallel "$(sysctl -n hw.ncpu)"

open "build-macos-arm64/ScanTailor Advanced for Mac.app"
```

The build deploys Qt frameworks and plugins into the app and applies an ad-hoc
signature. Copy the `.app` to Applications if desired. Keep the app bundle
intact; do not move just the executable out of it.

This is currently a source-build port, not a Developer ID signed or notarized
release. A successful local launch does not establish compatibility with every
macOS version. The minimum supported macOS version has not yet been established.

### Rebuilding after a Qt update

The deployment step overwrites existing bundled libraries so that the executable
and Qt frameworks stay in sync. Mixing a newly linked executable with older
bundled Qt libraries can cause a launch-time “Symbol not found” crash.

## Scope and testing

- Native arm64 app bundle and launch tested locally.
- No Intel or universal build is maintained.
- Full image-processing regression testing and clean-machine deployment testing
  remain outstanding.
- Existing settings identifiers are retained despite the Mac display-name change.

Please report Mac-port issues in
[this repository's issue tracker](https://github.com/ProbabilityEngineer/ScanTailor-Advanced-Mac/issues),
and distinguish them from issues reproduced with upstream.

## License

GNU General Public License version 3. See [LICENSE](LICENSE).
Upstream copyright and license notices are retained.
