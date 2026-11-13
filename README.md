# GFx binary SDK

Downloads are on the [Releases](../../releases) page: one archive per platform, with its SHA-256
in the release notes. This repository holds only the releases; GFx's source code is not public.

GFx is a render hardware interface: one C++ API over Direct3D 12, Vulkan, Metal and a Null
backend, switchable at runtime. Each archive holds everything needed to build against GFx without
its source code:

| Folder | Contents |
|---|---|
| `include/gfx/` | The public headers. `gfx/device.h` is the place to start; every type and function is documented there. |
| `bin/` (Windows) | `GFx.dll` (Release) and `GFxd.dll` (Debug), with stripped PDBs so call stacks name GFx's functions |
| `lib/` | Import libraries (Windows) or `libGFx.so` / `libGFx.dylib`, and the CMake package in `lib/cmake/GFx/` |
| `share/doc/GFx/` | The licence, and the notices of the third-party code compiled into GFx |

## Using it from CMake

```cmake
find_package(GFx 1.0 REQUIRED CONFIG)   # point CMAKE_PREFIX_PATH or GFx_DIR at this folder
target_link_libraries(my_game PRIVATE GFx::GFx)

# Slang shaders, compiled at build time into C++ headers (needs slangc from the Vulkan SDK or
# https://github.com/shader-slang/slang/releases).
gfx_compile_shaders(TARGET my_game SOURCES shaders/mesh.slang)
```

Or let CMake download it:

```cmake
include(FetchContent)
FetchContent_Declare(gfx_sdk
    URL      <the archive's release URL>
    URL_HASH SHA256=<its hash, from the release notes>)
FetchContent_MakeAvailable(gfx_sdk)
find_package(GFx 1.0 REQUIRED CONFIG PATHS "${gfx_sdk_SOURCE_DIR}" NO_DEFAULT_PATH)
```

**Windows:** GFx is a DLL. Copy it next to your program after each build:

```cmake
add_custom_command(TARGET my_game POST_BUILD
    COMMAND "${CMAKE_COMMAND}" -E copy_if_different "$<TARGET_FILE:GFx::GFx>" "$<TARGET_FILE_DIR:my_game>"
    VERBATIM)
```

CMake picks `GFxd.dll` for Debug builds and `GFx.dll` for the others. Debug and Release use
different C++ runtimes, so don't mix them.

## Compatibility

GFx's interface passes C++ standard-library types, so the application must use the same compiler
family and standard library as the build in this package:

| Package | Built with | Use with |
|---|---|---|
| `windows-x64` | MSVC (Visual Studio 2026), `/MD` runtime | Visual Studio 2022 or newer, x64 |
| `linux-x64` | GCC 14 on Ubuntu 24.04 | GCC 14+ or Clang with libstdc++, glibc 2.39+ |
| `macos-arm64` | Apple Clang (Xcode on GitHub's macos-latest) | macOS 13+ on Apple silicon |

From 1.0, GFx is source compatible across 1.x and binary compatible within a patch release: a
newer 1.0.x DLL can replace an older one without rebuilding, but a 1.1 package needs a rebuild.

## Licence

The GFx SDK License ([LICENSE.md](LICENSE.md), also in each archive at `share/doc/GFx/LICENSE.md`): free for noncommercial use, including shipping
the runtime libraries unmodified with your program. Commercial use needs a separate licence from
the author. GFx's source code is not published.
