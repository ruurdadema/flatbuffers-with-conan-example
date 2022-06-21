# Flatbuffers with Conan example

This repository demonstrates how flatbuffers and conan can be used together.

## Prerequisites

To build and run this project you will need to install the following external dependencies:

- Recent version of Git
- CMake (see CMakeLists.txt for required version)
- Conan (https://conan.io/downloads.html)
- Python 3.9 (https://www.python.org/downloads/)
- Windows: Visual Studio (Community | Professional | Enterprise) 2019
- macOS: XCode 12 or higher

## How to clone the project

```
git clone --recursive git@gitlab.com:ruurdadema/flatbuffers-with-conan-example.git
cd flatbuffers-with-conan-example
git submodule update --init --recursive
```

## How to build the project for macOS

```
# From the project root (generates build folder if necessary)
cmake -B build -GXcode -DCMAKE_BUILD_TYPE=Release -DCMAKE_OSX_DEPLOYMENT_TARGET=10.13
cmake --build build --config Release
```
