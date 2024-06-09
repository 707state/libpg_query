cmake_minimum_required(VERSION 3.10)
project(pg_query VERSION 5.1.0)

# Set the target and version variables
set(TARGET pg_query)
set(VERSION_MAJOR 5)
set(VERSION_MINOR 1)
set(VERSION_PATCH 0)

# Define source files
file(
  GLOB
  SRC_FILES
  "src/*.c"
  "src/postgres/*.c"
  "vendor/protobuf-c/protobuf-c.c"
  "vendor/xxhash/xxhash.c"
  "protobuf/pg_query.pb-c.c")

# Set output directories
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# Include directories
include_directories(. vendor src/include src/postgres/include)

# Compiler flags
set(CMAKE_C_FLAGS
    "${CMAKE_C_FLAGS} -Wall -Wno-unused-function -Wno-unused-value -Wno-unused-variable -fno-strict-aliasing -fwrapv -fPIC"
)

# Set OS-specific settings
if(APPLE)
  set(SOLIB lib${TARGET}.dylib)
  set(SONAME lib${TARGET}.${VERSION_MAJOR}.${VERSION_MINOR}.dylib)
  set(SOLIBVER
      lib${TARGET}.${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}.dylib)
else()
  set(SOLIB lib${TARGET}.so)
  set(SONAME ${SOLIB}.${VERSION_MAJOR}.${VERSION_MINOR})
  set(SOLIBVER ${SONAME}.${VERSION_PATCH})
endif()

# Optimization flags
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -D USE_ASSERT_CHECKING")
elseif(CMAKE_BUILD_TYPE STREQUAL "Release")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O3")
elseif(CMAKE_BUILD_TYPE STREQUAL "Valgrind")
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O0 -DUSE_VALGRIND")
endif()

# Define the library target
add_library(${TARGET} STATIC ${SRC_FILES})

# Set properties for the shared library
set_target_properties(
  ${TARGET}
  PROPERTIES VERSION ${VERSION_MAJOR}.${VERSION_MINOR}
             SOVERSION ${VERSION_MAJOR}
             OUTPUT_NAME ${TARGET})

# Installation rules
install(
  TARGETS ${TARGET}
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib)

install(FILES pg_query.h protobuf/pg_query.proto DESTINATION include/${TARGET})
