cmake_minimum_required(VERSION 3.21)
set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
if ("${CMAKE_SYSTEM_NAME}" STREQUAL "iOS" OR IOS OR GEODE_TARGET_PLATFORM STREQUAL "iOS")
    set(CMAKE_OSX_ARCHITECTURES "arm64")
else()
    set(CMAKE_OSX_ARCHITECTURES "arm64;x86_64")
endif()
set(CMAKE_CXX_VISIBILITY_PRESET hidden)

project(FrameInspector VERSION 1.0.0)

file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS src/*.cpp)
add_library(${PROJECT_NAME} SHARED ${SOURCES})

if (NOT DEFINED ENV{GEODE_SDK})
    message(FATAL_ERROR "Unable to find Geode SDK! Please define GEODE_SDK environment variable to point to Geode")
else()
    message(STATUS "Found Geode: $ENV{GEODE_SDK}")
endif()

add_subdirectory($ENV{GEODE_SDK} ${CMAKE_CURRENT_BINARY_DIR}/geode)

# The iOS standalone FMOD bindings are generated before any FMOD class
# headers. Geode 5.10.1 needs the C API declarations available at that point.
if (GEODE_TARGET_PLATFORM STREQUAL "iOS")
    target_compile_options(GeodeBindings PRIVATE
        "$<$<COMPILE_LANGUAGE:CXX>:SHELL:-include Geode/fmod/fmod.h>"
    )
endif()

setup_geode_mod(${PROJECT_NAME})
