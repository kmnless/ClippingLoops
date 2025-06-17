cmake_minimum_required(VERSION 3.15)
project(ClippingLoops VERSION 0.1 LANGUAGES CXX)

# Use C++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# --- Detect RtAudio ---
set(RTAUDIO_FOUND FALSE)

# 1) Try Config mode (e.g. vcpkg)
find_package(RtAudio CONFIG QUIET)
if (RtAudio_FOUND)
    message(STATUS "Found RtAudio via Config")
    set(RTAUDIO_INCLUDE_DIRS ${RtAudio_INCLUDE_DIRS})
    set(RTAUDIO_LIBS RtAudio::rtaudio)
    set(RTAUDIO_FOUND TRUE)
endif()

# 2) Fallback with find_path/find_library
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

# Error if not found
if (NOT RTAUDIO_FOUND)
    message(FATAL_ERROR "RtAudio not found!\n"
        "* Windows: install via vcpkg and pass -DCMAKE_TOOLCHAIN_FILE=<vcpkg>/scripts/buildsystems/vcpkg.cmake\n"
        "* Linux: install librtaudio-dev or set RTAUDIO_INCLUDE_DIR and RTAUDIO_LIB manually")
endif()

# --- Collect sources ---
file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS "src/*.cpp")
file(GLOB_RECURSE HEADERS CONFIGURE_DEPENDS "src/*.h")

# --- Create executable ---
add_executable(${PROJECT_NAME} ${SOURCES} ${HEADERS})

# --- Include and link RtAudio ---
target_include_directories(${PROJECT_NAME} PRIVATE ${RTAUDIO_INCLUDE_DIRS})
target_link_libraries(${PROJECT_NAME} PRIVATE ${RTAUDIO_LIBS})

# --- Platform-specific definitions ---
if (WIN32)
    target_compile_definitions(${PROJECT_NAME} PRIVATE PLATFORM_WINDOWS)
elseif (UNIX)
    target_compile_definitions(${PROJECT_NAME} PRIVATE PLATFORM_LINUX)
endif()
