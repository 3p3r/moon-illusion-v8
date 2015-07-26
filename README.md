# moon-illusion-v8
An easy-to-integrate CMake script to build [Google v8](https://developers.google.com/v8/)

## What's this?
This is a repo mainly created to be used in my personal projects and might come in handy to other people looking for a way to integrate v8 into their build systems as well.

## What does it do / why ?
It downloads a barebone clone of v8 and its dependencies and invokes `gyp` to generate `v8` project files. To my opinion the original build system of `v8` sucks and it is not easy to integrate at all. Since the time `v8` devs decided to rule out their own build system, `CMake` has evolved a lot and now it is almost always better than `gyp`.

## How to use it?
Just like a normal `CMakeLists.txt`. For example:
```CMD
cmake .. "Visual Studio 12"
```
If everything goes OK, you will find `v8` project files under a path where it will be logged by CMake.

## Settings and Dependencies
You need `python` on path and `setuptools` installed on it. There are two cache variables:

 - `V8_ROOT`: Where the source code of `v8` and all its dependencies will be stored
    - default: `${CMAKE_CURRENT_BINARY_DIR}/build_v8`
 - `V8_ARGS`: What will be passed to `gyp` as command line arguments.
    - default: `-Dv8_enable_i18n_support=0 -Dcomponent=static_library -Dv8_use_snapshot=0`

## Platform support
Only tested on Windows 7, 8, and 10 but most likely will work on other platforms as well.
