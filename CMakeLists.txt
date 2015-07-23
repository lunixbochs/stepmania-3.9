cmake_minimum_required(VERSION 2.6)

project(stepmania)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

add_definitions(-ggdb -ffast-math)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c99")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fpermissive")

include_directories(src)
add_definitions(-DENDIAN_BIG -Wno-deprecated)
file(GLOB SOURCES src/*.cpp src/crypto/*.cpp src/crypto51/*.cpp src/pcre/*.c)
foreach(item ${SOURCES})
    if(${item} MATCHES "RageDisplay" OR ${item} MATCHES "printint.c")
        list(REMOVE_ITEM SOURCES ${item})
    endif()
endforeach()

set(ARCH_SOURCES
    src/RageDisplay.cpp
    src/RageDisplay_Null.cpp
    src/arch/ArchHooks/ArchHooks.cpp
    src/arch/Dialog/Dialog.cpp
    src/arch/InputHandler/InputHandler.cpp
    src/arch/Lights/LightsDriver.cpp
    src/arch/Lights/LightsDriver_SystemMessage.cpp
    src/arch/MemoryCard/MemoryCardDriver.cpp
    src/arch/MovieTexture/MovieTexture.cpp
    src/arch/MovieTexture/MovieTexture_Null.cpp
    src/arch/Sound/RageSoundDriver_Generic_Software.cpp
    src/arch/Sound/RageSoundDriver_Null.cpp
    src/arch/arch.cpp
)

include_directories(src/arch)
if (APPLE)
    include_directories(src/archutils/Darwin/PublicUtility)
    list(APPEND SOURCES
        src/RageDisplay_OGL.cpp
        src/arch/ArchHooks/ArchHooks_darwin.cpp
        src/arch/InputHandler/InputHandler_SDL.cpp
        src/arch/LoadingWindow/LoadingWindow_SDL.cpp
        src/arch/LowLevelWindow/LowLevelWindow_SDL.cpp
        src/arch/Sound/RageSoundDriver_CA.cpp
        src/arch/Threads/Threads_Pthreads.cpp
    )
    add_definitions(-D_XOPEN_SOURCE)
    file(GLOB_RECURSE ARCHUTILS_SOURCES src/archutils/Darwin/*.cpp src/archutils/Unix/*cpp)
    foreach(item ${ARCHUTILS_SOURCES})
        if(${item} MATCHES "LinuxThreadHelpers")
            list(REMOVE_ITEM ARCHUTILS_SOURCES ${item})
        endif()
    endforeach()
    list(APPEND ARCH_SOURCES ${ARCHUTILS_SOURCES})
    add_definitions(-DDARWIN)
elseif (CMAKE_SYSTEM_NAME MATCHES "Linux")
    list(APPEND SOURCES src/RageDisplay_OGL.cpp)
    file(GLOB_RECURSE ARCHUTILS_SOURCES src/archutils/Unix/*.cpp)
    list(APPEND ARCH_SOURCES ${ARCHUTILS_SOURCES})
    add_definitions(-DLINUX)
else()
    message(FATAL_ERROR "Unsupported OS: ${CMAKE_SYSTEM_NAME}")
endif()

add_executable(stepmania ${SOURCES} ${ARCH_SOURCES})
target_link_libraries(stepmania SDL jpeg lua mad png resample vorbisfile z)

if (APPLE)
    find_library(AppKit AppKit)
    find_library(AudioToolbox AudioToolbox)
    find_library(CoreAudio CoreAudio)
    find_library(OpenGL OpenGL)
    target_link_libraries(stepmania ${AppKit} ${AudioToolbox} ${CoreAudio} ${OpenGL})
elseif (CMAKE_SYSTEM_NAME MATCHES "Linux")
    target_link_libraries(stepmania GL GLU X11)
endif()
