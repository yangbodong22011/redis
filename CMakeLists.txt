cmake_minimum_required (VERSION 2.8)
project (redis-unstable)

# C99 standard
set (CMAKE_C_STANDARD 99)

# use `cmake -DCMAKE_BUILD_TYPE=xxx`
if (NOT CMAKE_BUILD_TYPE)
    message(STATUS "No build type defined; defaulting to 'Debug'")
    set (CMAKE_BUILD_TYPE "Debug" CACHE STRING "The type of build. Possible values are: Debug, Release")
endif ()

message(STATUS "Host is: ${CMAKE_HOST_SYSTEM}.  Build target is: ${CMAKE_SYSTEM}")

# set ${REDIS_ROOT} is current dir
get_filename_component(REDIS_ROOT "${CMAKE_CURRENT_SOURCE_DIR}" ABSOLUTE)
message(STATUS "Project root directory is: ${REDIS_ROOT}")

if (CMAKE_SYSTEM_NAME MATCHES "Linux")
    set (CMAKE_C_COMPILER "gcc")
else ()
    set (CMAKE_C_COMPILER "clang")
endif ()

set (C_FLAGS
        -std=c99
        -DREDIS_STATIC=''
        -Wall
        -W
        -Wno-missing-field-initializers
        -Wno-unused-function
        -ggdb
        #    -DUSE_JEMALLOC
        )

if (CMAKE_SYSTEM_NAME MATCHES "Darwin")
    message(STATUS "MacOS? we must add SDK directory")
    include_directories("/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/include/")
    link_directories("/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk/usr/lib")
endif ()

if (CMAKE_C_COMPILER_ID STREQUAL "Clang")
    list (APPEND C_FLAGS "-Wno-null-dereference")
    list (APPEND C_FLAGS "-Wno-nullability-completeness")
    list (APPEND C_FLAGS "-Wno-nullability-extension")
    list (APPEND C_FLAGS "-Wno-expansion-to-defined")
    list (APPEND C_FLAGS "-Wno-macro-redefined")
    list (APPEND C_FLAGS "-Wno-gnu-zero-variadic-macro-arguments")
    list (APPEND C_FLAGS "-Wno-sign-conversion")
    list (APPEND C_FLAGS "-Wno-unused-local-typedef")
    list (APPEND C_FLAGS "-Wno-c11-extensions")
    list (APPEND C_FLAGS "-Wno-empty-translation-unit")
    list (APPEND C_FLAGS "-Wno-deprecated-declarations")
    list (APPEND C_FLAGS "-Wno-empty-body")
    list (APPEND C_FLAGS "-Wthread-safety")
    #    list (APPEND C_FLAGS "-fsanitize=address,undefined,null")
else ()
    list (APPEND C_FLAGS "-Wno-pedantic")
    list (APPEND C_FLAGS "-rdynamic")
endif ()

# Debug or Release flags
set (CMAKE_C_FLAGS_DEBUG "-O0")
set (CMAKE_C_FLAGS_RELEASE "-O2 -DNDEBUG")

# modify ${CMAKE_C_FLAGS} and show it
string(REPLACE ";" " " CMAKE_C_FLAGS "${C_FLAGS}")
message(STATUS "C_FLAGS = " ${CMAKE_C_FLAGS} " " ${CMAKE_C_FLAGS_${BUILD_TYPE}})

set (EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR})
set (LIBRARY_OUTPUT_PATH ${REDIS_ROOT}/modules)

add_subdirectory(deps)
add_subdirectory(src)
