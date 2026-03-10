# How-to-clangd-and-unity-build
A how-to for befriending clangd with unity builds for C/C++ devs. I tried my best on making this beginner-friendly, but if you don't understand something or want to make another suggestion, please go create an issue, so I could make it even more clear for others.

# Configuration file for clangd
Add `.clangd` file in your project directory. For example, I use two files:
```
...\projects folder
│   .clangd         <- common one
└───project-name
    │   .clangd     <- and project-specific
    ├───build
    │   └...
    └───code
        └...
```
For unity builds you need to make clangd `#include` your main source file or a corresponding header(you will need this header, so create it if you don't have it anyways). In this example main source and header are named "unity_build":
```
CompileFlags:
  Compiler: clang-cl
  Add: [-FIunity_build.cpp]
```
For `cl` compiler it's the same; for `clang`/`clang++`, use `-I...` instead; for other compilers, go see their docs.
If you are interested in changing clangd's behavior, see https://clangd.llvm.org/config.
# File structure
It's easier if you move out everything you can out of other headers in the "main" header. In the end of this header you should `#include` every other header of your project. You should get something like this:
```
#pragma once
#include <stdint.h>
// ...other standard/third-party headers...

using u8  = uint8_t;
// ...other typedefs...

#define local_persist   static
// ...other macros...

static constexpr f32 Pi32 = 3.141592653589793f;
// ...other constants... (I define constants with "static constexpr" instead of "#define"ing them)

// Basically everything that is not/can be not DEFINED in any of source files, move out here


// All first-party headers
#ifdef _WIN32
  #include "win32_handmade.h"
#endif // _WIN32
#include "handmade.h"
// BUT NO SOURCE FILES HERE!!!
```
Then you just `#include` this "main" header at the top of every source file. After that all includes are managed inside the main header. Don't forget that all your other headers, must have `#pragma once` too.
Then finally make a source file for this header that just includes all other sources:
```
#include "unity_build.h"

#ifdef _WIN32
  #include "win32_handmade.cpp"
#endif // _WIN32
#if HANDMADE_INTERNAL
  #include "handmade_internal.cpp"
#endif

#include "handmade.cpp"
```
# That's it!
Starting new project this way must be easier than migrating to this structure from other ones(even if it's just the same unity build without headers), but errors should guide you in either way, so it's not THAT hard.

You might think there are some redundancies, and in fact at first I had a simpler build(that didn't need header files). It worked for me while I used `clang++`, but somehow clangd couldn't do this with `clang-cl` that I decided to use later. But this must work for both I believe, although I haven't tested it on `clang++` again. I don't know how this even matters, and if there are compilers that don't work even with this method, but I believe for most circumstances this should work)
