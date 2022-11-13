cmake_minimum_required(VERSION 3.16...3.23)

# Not ideal to use this global variable, but necessary to make sure
# that tooling and projects use the same version.
set(CMAKE_CXX_STANDARD 20)

# Strongly encouraged to enable this globally to avoid conflicts between
# -Wpedantic being enabled and -std=c++20 and -std=gnu++20 for example
# when compiling with PCH enabled.
set(CMAKE_CXX_EXTENSIONS OFF)

# Note by default ENABLE_DEVELOPER_MODE is True
# This means that all analysis (sanitizers, static analysis)
# is enabled and all warnings are treated as errors.
# If you want to switch this behavior, change TRUE to FALSE.
set(ENABLE_DEVELOPER_MODE
    TRUE
    CACHE BOOL "Enable 'developer mode'")

#Change this to false if you want to disable warning_as_errors in developer mode
set(OPT_WARNINGS_AS_ERRORS_DEVELOPER_DEFAULT TRUE)

# Add project_options v0.26.1
# https://github.com/cpp-best-practices/project_options
include(FetchContent)
FetchContent_Declare(_project_options URL https://github.com/aminya/project_options/archive/refs/tags/v0.26.1.zip)
FetchContent_MakeAvailable(_project_options)
include(${_project_options_SOURCE_DIR}/Index.cmake)

# uncomment to enable vcpkg:
# # Setup vcpkg - should be called before defining project()
# run_vcpkg()

# Set the project name and language
project(
    pdCalc
    VERSION 0.0.1
    DESCRIPTION ""
    HOMEPAGE_URL "%%myurl%%"
    LANGUAGES CXX
)

# This variable is set by project() in CMake 3.21+
string(
    COMPARE EQUAL
        "${CMAKE_SOURCE_DIR}"
        "${PROJECT_SOURCE_DIR}"
        PROJECT_IS_TOP_LEVEL)

if(PROJECT_IS_TOP_LEVEL)
    # Consider CTest module, which creates targets and options!
    # Only needed if you want to enable submissions to a CDash server.
    include(CTest)
endif()

set(GIT_SHA
    "Unknown"
    CACHE STRING "SHA this build was generated from")

string(
    SUBSTRING "${GIT_SHA}"
              0
              8
              GIT_SHORT_SHA)

get_property(BUILDING_MULTI_CONFIG GLOBAL PROPERTY GENERATOR_IS_MULTI_CONFIG)

if(BUILDING_MULTI_CONFIG)
    if(NOT CMAKE_BUILD_TYPE)
        # Make sure that all supported configuration types have their
        # associated conan packages available. You can reduce this
        # list to only configuration types you use, but only if one
        # is not forced-set on the command line for VS.
        message(TRACE "Setting up multi-config build types.")
        set(CMAKE_CONFIGURATION_TYPES
            Debug
            Release
            RelWithDebInfo
            MinSizeRel
            CACHE STRING "Enable build types" FORCE)
    else()
        message(TRACE "User chose a specific build type, so we are using that.")
        set(CMAKE_CONFIGURATION_TYPES
            ${CMAKE_BUILD_TYPE}
            CACHE STRING "Enable build types" FORCE)
    endif()
endif()

include(${_project_options_SOURCE_DIR}/src/DynamicProjectOptions.cmake)

# default_project_options sets recommended defaults and provides user and developer
# modes and full GUI support for choosing options at configure time.

# for more flexibility, look into project_options() macro.

# Any default can be overridden.
# set(<feature_name>_DEFAULT <value>) - set default for both user and developer modes
# set(<feature_name>_DEVELOPER_DEFAULT <value>) - set default for developer mode
# set(<feature_name>_USER_DEFAULT <value> - set default for user mode)

# Initialize project_options variable related to this project.
# This overwrites `project_options` and sets `project_warnings`.
# Uncomment the options to enable them:
dynamic_project_options(
    # Note: PCH is disabled by default in developer mode because these headers become
    # globally included and they can mask other macros.
    PCH_HEADERS
    <vector>
    <string> # This is a list of headers to pre-compile, here are some common ones
    # ENABLE_CONAN
    # CONAN_OPTIONS     # Extra options to pass to conan
    # MSVC_WARNINGS     # Override the defaults for the MSVC warnings
    # CLANG_WARNINGS    # Override the defaults for the CLANG warnings
    # GCC_WARNINGS      # Override the defaults for the GCC warnings
    CPPCHECK_OPTIONS
    --enable=style,performance,warning,portability
    --inline-suppr
    # We cannot act on a bug/missing feature of cppcheck
    --supress=cppcheckError
    --supress=internalAstError
    # If a file does not have an internalAstError, we get an unmatchedSupression error
    --suppress=unmatchedSupression
    --suppress=passedByValue
    --supress=syntaxError
    --inconclusive
)
