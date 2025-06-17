cmake_minimum_required(VERSION 3.15)
project(ClippingLoops VERSION 0.1 LANGUAGES CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# --- Detect RtAudio library ---
set(RTAUDIO_FOUND FALSE)

# 1) Try to find a CMake Config package (e.g. installed via vcpkg)
find_package(RtAudio CONFIG QUIET)
if (RtAudio_FOUND)
    message(STATUS "Found RtAudio via Config")
    set(RTAUDIO_INCLUDE_DIRS ${RtAudio_INCLUDE_DIRS})
    set(RTAUDIO_LIBS RtAudio::rtaudio)
    set(RTAUDIO_FOUND TRUE)
endif()

# 2) Fallback: search manually in include and library paths
if (NOT RTAUDIO_FOUND)
    find_path(RTAUDIO_INCLUDE_DIR RtAudio.h)
    find_library(RTAUDIO_LIB NAMES rtaudio)
    if (RTAUDIO_INCLUDE_DIR AND RTAUDIO_LIB)
        message(STATUS "Found RtAudio via find_path/find_library")
        set(RTAUDIO_INCLUDE_DIRS ${RTAUDIO_INCLUDE_DIR})
        set(RTAUDIO_LIBS ${RTAUDIO_LIB})
        set(RTAUDIO_FOUND TRUE)
    endif()
endif()

# Error out if RtAudio still not found
if (NOT RTAUDIO_FOUND)
    message(FATAL_ERROR "RtAudio not found!\n"
        "* On Windows, install via vcpkg and pass -DCMAKE_TOOLCHAIN_FILE=<vcpkg>/scripts/buildsystems/vcpkg.cmake\n"
        "* On Linux, install librtaudio-dev (apt/pacman) or set RTAUDIO_INCLUDE_DIR and RTAUDIO_LIB manually")
endif()

# --- Collect all source and header files ---
file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS "src/*.cpp")
file(GLOB_RECURSE HEADERS CONFIGURE_DEPENDS "src/*.h")

# --- Define the executable target ---
add_executable(${PROJECT_NAME} ${SOURCES} ${HEADERS})

# --- Include RtAudio headers and link the library ---
target_include_directories(${PROJECT_NAME} PRIVATE ${RTAUDIO_INCLUDE_DIRS})
target_link_libraries(${PROJECT_NAME} PRIVATE ${RTAUDIO_LIBS})

# --- Platform-specific compiler definitions ---
if (WIN32)
    target_compile_definitions(${PROJECT_NAME} PRIVATE PLATFORM_WINDOWS)
elseif (UNIX)
    target_compile_definitions(${PROJECT_NAME} PRIVATE PLATFORM_LINUX)
endif()
