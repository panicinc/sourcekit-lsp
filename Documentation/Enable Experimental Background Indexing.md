# Background Indexing

Background indexing in SourceKit-LSP is available as an experimental feature. This guide shows how to set up background indexing and which caveats to expect.

## Set Up

1. Install a `main` or `release/6.0` Swift Development Snapshot from https://www.swift.org/install.
2. Point your editor to the newly installed toolchain.
   - In VS Code on macOS this can be done by adding the following to your `settings.json`: `"swift.path": "/Library/Developer/Toolchains/swift-latest.xctoolchain/usr/bin"`
   - In VS Code on other platforms, you need to set the `swift.path` to the `usr/bin` directory of your toolchain’s install location.
   - Other editors likely also have a way to pick the Swift toolchain, the exact steps vary by your setup.
3. Enable the experimental `background-indexing` feature.
   - In VS Code, add the following to your `settings.json`: `"swift.sourcekit-lsp.serverArguments": [ "--experimental-feature", "background-indexing" ]`
   - In other editors, change the invocation that launches `sourcekit-lsp` to pass the `--experimental-feature background-indexing` command line arguments. The exact steps vary by your setup.

## Known issues

- Background Indexing is only supported for SwiftPM projects [#1269](https://github.com/swiftlang/sourcekit-lsp/issues/1269), [#1271](https://github.com/swiftlang/sourcekit-lsp/issues/1271)
- If a module or one of its dependencies has a compilation error, it cannot be properly prepared for indexing because we are running a regular `swift build` to generate its modules [#1254](https://github.com/swiftlang/sourcekit-lsp/issues/1254) rdar://128683404
  - Workaround 1: Ensure that your files dependencies are in a buildable state to get an up-to-date index and proper cross-module functionality
  - Workaround 2: Enable the `swiftpm-prepare-for-indexing` experimental feature, which continues to build Swift module even in the presence of errors.
- If you change a function in a way that changes its USR but keeps it API compatible (such as adding a defaulted parameter), references to it will be lost and not re-indexed automatically [#1264](https://github.com/swiftlang/sourcekit-lsp/issues/1264)
  - Workaround: Make some edit to the files that had references to re-index them
- The index build is currently completely separate from the command line build generated using `swift build`. Building *does not* update the index (break your habits of always building!) [#1270](https://github.com/swiftlang/sourcekit-lsp/issues/1270)
- The initial indexing might take 2-3x more time than a regular build [#1254](https://github.com/swiftlang/sourcekit-lsp/issues/1254), [#1262](https://github.com/swiftlang/sourcekit-lsp/issues/1262), [#1268](https://github.com/swiftlang/sourcekit-lsp/issues/1268)
- Spurious re-indexing of ~10-20 source files when `swift build` writes a header to the build directory [rdar://128573306](rdar://128573306)

## Filing issues

If you hit any issues that are not mentioned above, please [file a GitHub issue](https://github.com/swiftlang/sourcekit-lsp/issues/new/choose) and attach the following information, if possible:
- A diagnostic bundle generated by running `path/to/sourcekit-lsp diagnose`.
- Your project including the `.index-build` folder, if possible.
