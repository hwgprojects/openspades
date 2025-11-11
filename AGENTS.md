# Repository Guidelines

## Project Structure & Module Organization
Source lives in `Sources/` (Client/Core/Gui/Audio modules, each with its own `CMakeLists.txt`). Build helpers reside in `cmake/`, shared assets in `Resources/`, and packaging specs in `flatpak/`, `snap/`, `OpenSpades.xcodeproj`, and `XSpades`. Keep build artifacts in out-of-tree directories such as `build/` or `openspades.mk` to avoid polluting the tree.

## Build, Test, and Development Commands
- Desktop build: `cmake -S . -B build -DCMAKE_BUILD_TYPE=RelWithDebInfo && cmake --build build -j`, then launch via `build/bin/openspades`.
- Windows 32-bit toolchain: run `vcpkg\bootstrap-vcpkg.bat`, `vcpkg\vcpkg install @vcpkg_x86-windows.txt`, then `cmake -S . -B OpenSpades.msvc -G "Visual Studio 17 2022" -A Win32 -DCMAKE_TOOLCHAIN_FILE=vcpkg/scripts/buildsystems/vcpkg.cmake -DVCPKG_TARGET_TRIPLET=x86-windows-static` and build inside Visual Studio.
- Reproducible shell: `nix develop` (or `nix-shell`) pins the compilers and tools defined in `flake.nix`.
- Smoke tests: `ctest --output-on-failure -C RelWithDebInfo` in your build tree, followed by `bin/openspades` after copying `Resources/`.

## Coding Style & Naming Conventions
`.clang-format` mandates tabs, Allman braces, and aligned pointer qualifiers; run `clang-format -i` or `pwsh ./run-clang-format.ps1` before committing. `.editorconfig` enforces LF endings and UTF-8 scripts. Use PascalCase for types, camelCase for functions/fields, SCREAMING_CASE for constants, and prefer established `spades::` namespaces. Keep headers self-contained and minimize global state.

## Testing Guidelines
Automated coverage is sparse, so prioritize integration passes: run the client against a local PySnip server, verify audio (`openal32.dll`) and optional `Nonfree` pak assets, and confirm resource loading on Linux and Windows. When adding deterministic code (math, serialization, file IO), drop focused tests next to the module and register them with CTest. Document manual test steps in your PR so reviewers can reproduce.

## Commit & Pull Request Guidelines
History uses a Conventional Commit flavor (`fix(gui): …`, `chore(ci): … (#1099)`); keep commits scoped and buildable. PRs should explain motivation, list build/test evidence, mention asset or pak changes, and attach screenshots for UI tweaks. Update README or TROUBLESHOOTING entries when behavior changes and ensure submodules, Crowdin strings, or vcpkg locks stay synchronized.

## Localization & Asset Notes
Translations sync through Crowdin (`crowdin.yml`); run `./crowdin-download.sh` before editing UI text. Non-free art belongs outside source control except for the documented `Resources/Nonfree` drop. When touching shaders, sounds, or pak metadata, call out the affected files so Snap/Flatpak maintainers can rebuild their bundles.
