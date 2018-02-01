#  :copyright: (c) 2017-2018 Alex Huszagh.
#  :license: MIT, see licenses/mit.md for more details.

cmake_minimum_required(VERSION 3.1.0)
project(hashxx C CXX)
enable_testing()

# OPTIONS
option(HASHXX_BUILD_TESTS "Build hashxx library tests" OFF)

# STATIC/SHARED
if(NOT BUILD_SHARED_LIBS)
    set(BUILD_STATIC_LIBS OFF)
    if(MINGW OR MSYS OR CYGWIN)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -static")
    elseif(UNIX)
        set(CMAKE_POSITION_INDEPENDENT_CODE ON)
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
    endif()
endif()

set(HASHXX_HEADERS
    ${CMAKE_CURRENT_SOURCE_DIR}/src/hashxx.h
    ${CMAKE_CURRENT_SOURCE_DIR}/src/hashxx/xxhash.h
)

set(HASHXX_SOURCE_FILES
    ${CMAKE_CURRENT_SOURCE_DIR}/src/hashxx/xxhash.c
)

set(HASHXX_TESTS
    test/hashxx.cc
)

# LIBRARY
add_library(hashxx "")
target_include_directories(hashxx INTERFACE src)
target_sources(hashxx
    PRIVATE ${HASHXX_SOURCE_FILES}
    INTERFACE ${HASHXX_HEADERS}
)

if(HASHXX_BUILD_TESTS)
    include(GoogleTest)
    add_executable(hashxx_tests ${HASHXX_TESTS})
    target_link_libraries(hashxx_tests
        hashxx
        gtest
        gtest_main
    )
    add_test(NAME hashxx_tests COMMAND hashxx_tests)
endif()

# INSTALL
install(DIRECTORY src/
    DESTINATION include
    FILES_MATCHING PATTERN "*.h"
)

# UNINSTALL
if(NOT TARGET uninstall)
    set(UNINSTALL_FILE ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake)
    set(UNINSTALL_FILE_IN ${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in)
    configure_file("${UNINSTALL_FILE_IN}" "${UNINSTALL_FILE}" IMMEDIATE @ONLY)

    add_custom_target(uninstall COMMAND ${CMAKE_COMMAND} -P ${UNINSTALL_FILE})
endif()
