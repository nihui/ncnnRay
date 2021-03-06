cmake_minimum_required(VERSION 3.0)
set(CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

set(CPP11_EXTRA_FLAGS "")
list(APPEND CMAKE_CXX_FLAGS " -pthread  ${CPP11_EXTRA_FLAGS} ${CMAKE_CXX_FLAGS}")
set(CMAKE_EXE_LINKER_FLAGS "-pthread ${CMAKE_EXE_LINKER_FLAGS} ${CPP11_EXTRA_FLAGS}")

# Config options
option(BUILD_EXAMPLES "Build the examples." OFF)
option(ENABLE_ASAN  "Enable AddressSanitizer (ASAN) for debugging (degrades performance)" OFF)
option(ENABLE_UBSAN "Enable UndefinedBehaviorSanitizer (UBSan) for debugging" OFF)
option(ENABLE_MSAN "Enable MemorySanitizer (MSan) for debugging (not recommended to run with ASAN)" OFF)

if(CMAKE_VERSION VERSION_LESS "3.1")
  if(CMAKE_C_COMPILER_ID STREQUAL "GNU")
    set(CMAKE_C_FLAGS "-std=gnu99 ${CMAKE_C_FLAGS}")
  endif()
else()
  set (CMAKE_C_STANDARD 99)
endif()

include(AddIfFlagCompiles)
add_if_flag_compiles(-Werror=pointer-arith CMAKE_C_FLAGS)
add_if_flag_compiles(-Werror=implicit-function-declaration CMAKE_C_FLAGS)
# src/external/jar_xm.h does shady stuff
add_if_flag_compiles(-fno-strict-aliasing CMAKE_C_FLAGS)

if (ENABLE_ASAN)
    add_if_flag_compiles(-fno-omit-frame-pointer CMAKE_C_FLAGS CMAKE_LINKER_FLAGS)
    add_if_flag_compiles(-fsanitize=address CMAKE_C_FLAGS CMAKE_LINKER_FLAGS)
endif()
if (ENABLE_UBSAN)
    add_if_flag_compiles(-fno-omit-frame-pointer CMAKE_C_FLAGS CMAKE_LINKER_FLAGS)
    add_if_flag_compiles(-fsanitize=undefined CMAKE_C_FLAGS CMAKE_LINKER_FLAGS)
endif()
if (ENABLE_MSAN)
    add_if_flag_compiles(-fno-omit-frame-pointer CMAKE_C_FLAGS CMAKE_LINKER_FLAGS)
    add_if_flag_compiles(-fsanitize=memory CMAKE_C_FLAGS CMAKE_LINKER_FLAGS)
endif()

if (ENABLE_MSAN AND ENABLE_ASAN)
    MESSAGE(WARNING "Compiling with both AddressSanitizer and MemorySanitizer is not recommended")
endif()

add_subdirectory(src)


if (${BUILD_EXAMPLES})
  add_subdirectory(examples)
endif()

enable_testing()
