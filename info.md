# Build Notes (2025-11-11)

## Tooling
- Visual Studio Build Tools 2022 (v143 toolset) and Kitware CMake 3.26.4 are installed; ensure both remain on PATH for future builds.
- `vcpkg` submodule has been updated to commit `74e6536` so dependency detection works with VS2022; packages installed for `x86-windows-static`.

## SDL2_image Alias Fix
Because OpenSpades expects `SDL2::SDL2_image` while modern vcpkg exports `SDL2_image::SDL2_image-static`, two tiny shims were added:
1. `vcpkg/installed/x86-windows-static/share/sdl2-image/sdl2-imageConfig.cmake` and `…-config.cmake` now include the canonical config from `../SDL2_image/`.
2. `vcpkg/installed/x86-windows-static/share/SDL2_image/SDL2_imageConfig.cmake` appends an alias block that creates `SDL2::SDL2_image` if it is missing. Reinstalling the port will overwrite these snippets, so reapply them if you wipe `installed/`.

## Asset Builder Quoting
`Resources/CMakeLists.txt` now calls PowerShell with `-File "<path>" "<binary-dir>"`, allowing `cmake --build …` to succeed even when the repo lives under a path containing spaces (e.g., `F:\Proton Drive\…`). Upstream should eventually receive the same quoting fix.

## Build / Run Recipe
```pwsh
$env:Path = 'C:\Program Files\CMake\bin;' + $env:Path
cmake -S . -B build -G "Visual Studio 17 2022" -A Win32 `
  -DCMAKE_TOOLCHAIN_FILE=vcpkg/scripts/buildsystems/vcpkg.cmake `
  -DVCPKG_TARGET_TRIPLET=x86-windows-static -DCMAKE_BUILD_TYPE=RelWithDebInfo
cmake --build build --config RelWithDebInfo --parallel
```
The resulting executable and resources live in `build/bin/RelWithDebInfo/`. Launch `OpenSpades.exe` from there; add optional nonfree `.pak` files under `build/bin/RelWithDebInfo/Resources` if needed.

## Outstanding Warnings
- MSVC still emits a series of `C4244`/`C4305` float truncation warnings within `Sources/Client/*.cpp`; these are pre-existing upstream and do not break the build.
- CMake warns about deprecated minimum versions (2.8.x). Updating the top-level `cmake_minimum_required` and nested AngelScript CMake files would silence them if desired.
