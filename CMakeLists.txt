cmake_minimum_required(VERSION 3.21)

# Configure C/CXX
set(CMAKE_C_STANDARD 17)
set(CMAKE_C_STANDARD_REQUIRED True)
set(CMAKE_C_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_CXX_EXTENSIONS OFF)

# Enable folders
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# Add project wide cmake scripts
add_subdirectory(cmake)

# Git version
find_package(Git)

set(GIT_TAG "0.0.1")
set(GIT_SHA "Unknown")

if(GIT_FOUND)
    execute_process(
        COMMAND ${GIT_EXECUTABLE} describe --always --first-parent --abbrev=8 --match '*.*.*'
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        OUTPUT_VARIABLE GIT_DESCRIPTION
        OUTPUT_STRIP_TRAILING_WHITESPACE
        ECHO_ERROR_VARIABLE
        COMMAND_ERROR_IS_FATAL ANY
    )

    string(REPLACE "-" ";" GIT_DESCRIPTION "${GIT_DESCRIPTION}")
    list(LENGTH GIT_DESCRIPTION GIT_DESCRIPTION_LENGTH)

    if(GIT_DESCRIPTION_LENGTH GREATER 1)
        list(GET GIT_DESCRIPTION 0 GIT_TAG)
        list(GET GIT_DESCRIPTION 1 GIT_SHA)
    else()
        list(GET GIT_DESCRIPTION 0 GIT_SHA)
    endif()
endif()

# Configure project
project(
    nifly-utils
    VERSION ${GIT_TAG}
    DESCRIPTION "C++ library that adds additional utility functions to the wonderful nifly library."
    HOMEPAGE_URL "https://github.com/SK83RJOSH/nifly-utils"
    LANGUAGES CXX C
)

if(PROJECT_IS_TOP_LEVEL)
    include(CTest)
endif()

set(CLANG CMAKE_CXX_COMPILER_ID MATCHES "Clang" OR CMAKE_C_COMPILER_ID MATCHES "Clang")

# Configure multi-config
get_property(BUILDING_MULTI_CONFIG GLOBAL PROPERTY GENERATOR_IS_MULTI_CONFIG)
if(BUILDING_MULTI_CONFIG)
  if(NOT CMAKE_BUILD_TYPE)
    message(TRACE "Setting up multi-config build types")
    set(CMAKE_CONFIGURATION_TYPES
        Debug
        Release
        RelWithDebInfo
        MinSizeRel
        CACHE STRING "Enabled build types" FORCE)
  else()
    message(TRACE "User chose a specific build type, so we are using that")
    set(CMAKE_CONFIGURATION_TYPES
        ${CMAKE_BUILD_TYPE}
        CACHE STRING "Enabled build types" FORCE)
  endif()
endif()

# Configure validation
option(OPT_ENABLE_VALIDATION "Enable validation (static analysis, coverage, and sanitizers)" OFF)

# Configure static analysis
set(ENABLE_CLANG_TIDY OFF)
set(ENABLE_CPPCHECK OFF)
set(ENABLE_VS_ANALYSIS OFF)

option(OPT_ENABLE_STATIC_ANALYSIS "Enable static analysis" ${OPT_ENABLE_VALIDATION})

if(OPT_ENABLE_STATIC_ANALYSIS)
    set(ENABLE_CLANG_TIDY "ENABLE_CLANG_TIDY")
    set(ENABLE_CPPCHECK "ENABLE_CPPCHECK")
    set(ENABLE_VS_ANALYSIS "ENABLE_VS_ANALYSIS")
endif()

# Configure coverage
set(ENABLE_COVERAGE OFF)

option(OPT_ENABLE_COVERAGE "Enable coverage" ${OPT_ENABLE_VALIDATION})

if(OPT_ENABLE_COVERAGE)
    set(ENABLE_COVERAGE "ENABLE_COVERAGE")
endif()

# Configure doxygen
set(ENABLE_DOXYGEN OFF)
set(DOXYGEN_THEME "awesome-sidebar")

option(OPT_ENABLE_DOXYGEN "Enable doxygen" OFF)

if(OPT_ENABLE_DOXYGEN)
    set(ENABLE_DOXYGEN "ENABLE_DOXYGEN")
endif()

# Configure sanitizers
set(ENABLE_SANITIZER_ADDRESS OFF)
set(ENABLE_SANITIZER_LEAK OFF)
set(ENABLE_SANITIZER_UNDEFINED_BEHAVIOR OFF)

option(OPT_ENABLE_SANITIZE "Enable sanitizers" ${OPT_ENABLE_VALIDATION})

if(OPT_ENABLE_SANITIZE)
    # Can't use clang's address sanitizers on windows
    if(NOT CLANG OR NOT WIN32)
        set(ENABLE_SANITIZER_ADDRESS "ENABLE_SANITIZER_ADDRESS")
        set(ENABLE_SANITIZER_LEAK "ENABLE_SANITIZER_LEAK")
    endif()
    set(ENABLE_SANITIZER_UNDEFINED_BEHAVIOR "ENABLE_SANITIZER_UNDEFINED_BEHAVIOR")
endif()

# Configure include-what-you-use
set(ENABLE_INCLUDE_WHAT_YOU_USE OFF)

option(OPT_ENABLE_INCLUDE_WHAT_YOU_USE "Enable include-what-you-use" OFF)

if(OPT_ENABLE_INCLUDE_WHAT_YOU_USE)
    set(ENABLE_INCLUDE_WHAT_YOU_USE "ENABLE_INCLUDE_WHAT_YOU_USE")
endif()

# Configure PCH
set(ENABLE_PCH ON)

# Can't combine PCH with CLANG_TIDY on non-llvm compilers
if(ENABLE_CLANG_TIDY AND NOT CLANG)
    set(ENABLE_PCH OFF)
endif()

# Initialize project
project_options(
    ENABLE_CACHE
    ${ENABLE_CPPCHECK}
    CPPCHECK_OPTIONS
        --enable=style,performance,warning,portability
        --inline-suppr
        --suppress=cppcheckError
        --suppress=internalAstError
        --suppress=unmatchedSuppression
        --suppress=passedByValue
        --suppress=syntaxError
        --inconclusive
    ${ENABLE_CLANG_TIDY}
    ${ENABLE_VS_ANALYSIS}
    ${ENABLE_COVERAGE}
    ${ENABLE_DOXYGEN}
    WARNINGS_AS_ERRORS
    ${ENABLE_SANITIZER_ADDRESS}
    ${ENABLE_SANITIZER_LEAK}
    ${ENABLE_SANITIZER_UNDEFINED_BEHAVIOR}
    ${ENABLE_PCH}
    PCH_HEADERS
        <algorithm>
        <array>
        <codecvt>
        <deque>
        <filesystem>
        <fstream>
        <functional>
        <iomanip>
        <istream>
        <locale>
        <map>
        <mutex>
        <optional>
        <ostream>
        <regex>
        <set>
        <span>
        <string_view>
        <string>
        <tuple>
        <unordered_map>
        <unordered_set>
        <variant>
        <vector>
    ${ENABLE_INCLUDE_WHAT_YOU_USE}
)

# Add external dependencies
add_subdirectory(external)

target_disable_static_analysis(nifly)
target_disable_static_analysis(DirectXTex)

# Configure nifly-utils
set(NIFLY_UTILS_DIR ${CMAKE_CURRENT_SOURCE_DIR})
set(NIFLY_UTILS_TESTS_DIR ${NIFLY_UTILS_DIR}/tests)
set(NIFLY_UTILS_INCLUDE_DIR ${NIFLY_UTILS_DIR}/include)
set(NIFLY_UTILS_SOURCE_DIR ${NIFLY_UTILS_DIR}/source)
set(NIFLY_UTILS_EXTERNAL_DIR ${NIFLY_UTILS_DIR}/external)
set(NIFLY_UTILS_DEPENDENCIES nifly DirectXTex)

file(GLOB_RECURSE NIFLY_UTILS_HEADERS CONFIGURE_DEPENDS ${NIFLY_UTILS_INCLUDE_DIR}/*.hpp)
file(GLOB_RECURSE NIFLY_UTILS_SOURCES CONFIGURE_DEPENDS ${NIFLY_UTILS_SOURCE_DIR}/*.cpp)

add_library(nifly-utils STATIC ${NIFLY_UTILS_SOURCES} ${NIFLY_UTILS_HEADERS})
target_include_directories(
    nifly-utils PUBLIC
    $<BUILD_INTERFACE:${NIFLY_UTILS_INCLUDE_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}/nifly-utils>
)
target_link_libraries(nifly-utils PRIVATE project_options project_warnings)
target_link_system_libraries(nifly-utils PRIVATE ${NIFLY_UTILS_DEPENDENCIES})

# Merge files from source and include in the IDE
function(group_files SOURCES)
    foreach(FILE ${SOURCES})
        # Get the directory of the source file
        get_filename_component(PARENT_DIR "${FILE}" DIRECTORY)

        # Remove common directory prefix to make the group
        string(REPLACE "${NIFLY_UTILS_DIR}" "" GROUP "${PARENT_DIR}")

        # Make sure we are using windows slashes
        string(REPLACE "/" "\\" GROUP "${GROUP}")

        # Strip the root parts for each possible component
        if("${FILE}" MATCHES "include/.*")
            string(SUBSTRING ${GROUP} 8 -1 GROUP)
        elseif("${FILE}" MATCHES "source/.*")
            string(SUBSTRING ${GROUP} 7 -1 GROUP)
        endif()

        # Do the grouping
        source_group("${GROUP}" FILES "${FILE}")
    endforeach()
endfunction()
group_files("${NIFLY_UTILS_SOURCES};${NIFLY_UTILS_HEADERS}")

# Configure install
if(CMAKE_SKIP_INSTALL_RULES)
  return()
elseif(NOT PROJECT_IS_TOP_LEVEL)
  return()
endif()

package_project(TARGETS nifly-utils project_options project_warnings ${NIFLY_UTILS_DEPENDENCIES})

# Configure CPack
set(
    CPACK_PACKAGE_FILE_NAME
    "${CMAKE_PROJECT_NAME}-${CMAKE_PROJECT_VERSION}-${GIT_SHORT_SHA}-${CMAKE_SYSTEM_NAME}-${CMAKE_BUILD_TYPE}-${CMAKE_CXX_COMPILER_ID}-${CMAKE_CXX_COMPILER_VERSION}"
)

include(CPack)
