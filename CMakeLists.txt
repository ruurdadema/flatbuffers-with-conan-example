cmake_minimum_required(VERSION 3.15)

project(FlatbuffersDemo VERSION 0.0.1)

set(CMAKE_CXX_STANDARD 17)

if (NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Debug)
    message(STATUS "CMAKE_BUILD_TYPE not specified, set it to 'Debug'")
endif ()

# Conan

list(APPEND CMAKE_MODULE_PATH ${CMAKE_BINARY_DIR})
list(APPEND CMAKE_PREFIX_PATH ${CMAKE_BINARY_DIR})

include(submodules/cmake-conan/conan.cmake)

# When cross compiling (between macOS & iOS (iPadOS, iPhoneOS) or between architecture (Intel & ARM) we need the flatc
# executable still to be compiled for the host platform, as the host platform is going to need to execute that. In order
# to make sure that the flatc is built for the host platform, we create a separate flatc executable and pass that to the
# flatbuffers cmake tools later on.
conan_cmake_configure(
        REQUIRES flatbuffers/2.0.5
        GENERATORS cmake_find_package
        IMPORTS "bin, flatc -> ./bin"
        IMPORTS "bin, flatc.exe -> ./bin"
        NO_OUTPUT_DIRS
)
conan_cmake_autodetect(settings)
conan_cmake_install(PATH_OR_REFERENCE .
        BUILD missing
        SETTINGS ${settings}
        )

if (CMAKE_OSX_DEPLOYMENT_TARGET)
    list(APPEND settings os.version=${CMAKE_OSX_DEPLOYMENT_TARGET})
endif ()

if (CMAKE_OSX_ARCHITECTURES)
    if (${CMAKE_OSX_ARCHITECTURES} STREQUAL "arm64")
        list(APPEND settings arch=armv8)
    else ()
        list(APPEND settings arch=${CMAKE_OSX_ARCHITECTURES})
    endif ()
endif ()

conan_cmake_install(PATH_OR_REFERENCE ${CMAKE_CURRENT_SOURCE_DIR}/conanfile.txt
        BUILD missing
        REMOTE conancenter
        SETTINGS ${settings}
        )

find_package(FlatBuffers)
find_package(spdlog)
find_package(fmt)

# Flatbuffers
message(STATUS "Flatbuffer schemas: ${FLATBUFFER_SCHEMAS}")

if (WIN32)
    set(FLATBUFFERS_FLATC_EXECUTABLE ${CMAKE_BINARY_DIR}/bin/flatc.exe)
else ()
    # Set the just built flatc executable as flatc to be used by the flatbuffers cmake tool.
    set(FLATBUFFERS_FLATC_EXECUTABLE ${CMAKE_BINARY_DIR}/bin/flatc)
endif ()

flatbuffers_generate_headers(
        TARGET FlatbuffersGeneratedHeaders
        SCHEMAS source/HelloWorld.fbs
        FLAGS --gen-object-api --reflect-names --gen-name-strings --gen-compare
)
include_directories(${CMAKE_BINARY_DIR}/FlatbuffersGeneratedHeaders)

add_library(FlatbuffersGenerated STATIC source/EmptySource.cpp)
target_link_libraries(FlatbuffersGenerated FlatbuffersGeneratedHeaders flatbuffers::flatbuffers)

add_executable(TestFlatbuffers source/Main.cpp)
target_link_libraries(TestFlatbuffers PUBLIC
        FlatbuffersGenerated
        spdlog::spdlog
        fmt::fmt
        )
