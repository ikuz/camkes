#
# Copyright 2018, Data61
# Commonwealth Scientific and Industrial Research Organisation (CSIRO)
# ABN 41 687 119 230.
#
# This software may be distributed and modified according to the terms of
# the BSD 2-Clause license. Note that NO WARRANTY is provided.
# See "LICENSE_BSD2.txt" for details.
#
# @TAG(DATA61_BSD)
#

cmake_minimum_required(VERSION 3.8.2)

find_file(
    CAMKES_CMAKE camkes.cmake
    PATHS ${CMAKE_SOURCE_DIR}/projects/camkes-tool/
    CMAKE_FIND_ROOT_PATH_BOTH
)
mark_as_advanced(FORCE CAMKES_CMAKE)
if("${CAMKES_CMAKE}" STREQUAL "CAMKES_CMAKE-NOTFOUND")
    message(
        FATAL_ERROR
            "Failed to find camkes.cmake. Consider cmake -DCAMKES_CMAKE=/path/to/camkes.cmake"
    )
endif()
include(${CAMKES_CMAKE})

find_file(
    GLOBAL_COMPONENTS_PATH global-components.cmake
    PATHS ${CMAKE_SOURCE_DIR}/projects/global-components/
    CMAKE_FIND_ROOT_PATH_BOTH
)
mark_as_advanced(FORCE GLOBAL_COMPONENTS_PATH)
if("${GLOBAL_COMPONENTS_PATH}" STREQUAL "GLOBAL_COMPONENTS_PATH-NOTFOUND")
    message(
        FATAL_ERROR
            "Failed to find global-components.cmake. Consider cmake -DGLOBAL_COMPONENTS_PATH=/path/to/global-components.cmake"
    )
endif()

function(includeGlobalComponents)
    include(${GLOBAL_COMPONENTS_PATH})
endfunction()

# figure out the valid apps
set(app_names "")
file(GLOB apps apps/*)
foreach(ARG ${apps})
    get_filename_component(filename ${ARG} NAME)
    list(APPEND app_names "${filename}")
endforeach()
string(
    REPLACE
        ";"
        "\n  "
        app_names_error
        "${app_names}"
)

if("${CAMKES_APP}" STREQUAL "")
    message(
        FATAL_ERROR "Missing option -DCAMKES_APP=<app> to build. Valid apps:\n  ${app_names_error}"
    )
endif()

list(FIND app_names "${CAMKES_APP}" app_exists)
if(${app_exists} EQUAL -1)
    message(
        FATAL_ERROR
            "Invalid value for option -DCAMKES_APP=${CAMKES_APP}. Valid options:\n  ${app_names_error}"
    )
endif()

# We only support one application being built at a time
include(apps/${CAMKES_APP}/CMakeLists.txt)

# Should be done adding targets, can now generate the root server and the global configuration
GenerateCAmkESRootserver()
if(SIMULATION)
    GenerateSimulateScript()
endif()
