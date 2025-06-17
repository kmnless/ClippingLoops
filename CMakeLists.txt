cmake_minimum_required(VERSION 3.15)
project(ClippingLoops VERSION 0.1 LANGUAGES CXX)

# 1) C++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 2) Detect RtAudio
set(RTAUDIO_FOUND FALSE)

if (WIN32)
  # Windows: use vcpkg‑provided Config
  find_package(RtAudio CONFIG QUIET)
  if (RtAudio_FOUND)
    message(STATUS "Found RtAudio via vcpkg Config")
    set(RTAUDIO_INCLUDE_DIRS ${RtAudio_INCLUDE_DIRS})
    set(RTAUDIO_LIBS RtAudio::rtaudio)
    set(RTAUDIO_FOUND TRUE)
  endif()

elseif(UNIX)
  # Linux: use pkg-config
  find_package(PkgConfig REQUIRED)
  pkg_check_modules(RTAUDIO_PKG QUIET rtaudio)
  if (RTAUDIO_PKG_FOUND)
    message(STATUS "Found RtAudio via pkg-config")
    set(RTAUDIO_INCLUDE_DIRS ${RTAUDIO_PKG_INCLUDE_DIRS})
    set(RTAUDIO_LIBS       ${RTAUDIO_PKG_LIBRARIES})
    set(RTAUDIO_FOUND TRUE)
  endif()
endif()

# 3) Fatal if still not found
if (NOT RTAUDIO_FOUND)
  message(FATAL_ERROR
    "RtAudio not found!\n"
    "• Windows: install with vcpkg and use -DCMAKE_TOOLCHAIN_FILE=.../vcpkg.cmake\n"
    "• Linux: install rtaudio and pkgconf (Arch: sudo pacman -S rtaudio pkgconf)\n"
    "  then rerun CMake.")
endif()

# 4) Sources
file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS "src/*.cpp")
file(GLOB_RECURSE HEADERS CONFIGURE_DEPENDS "src/*.h")

# 5) Executable
add_executable(${PROJECT_NAME} ${SOURCES} ${HEADERS})

# 6) Link
target_include_directories(${PROJECT_NAME} PRIVATE ${RTAUDIO_INCLUDE_DIRS})
target_link_libraries(${PROJECT_NAME} PRIVATE ${RTAUDIO_LIBS})

# 7) Platform define
if (WIN32)
  target_compile_definitions(${PROJECT_NAME} PRIVATE PLATFORM_WINDOWS)
else()
  target_compile_definitions(${PROJECT_NAME} PRIVATE PLATFORM_LINUX)
endif()
