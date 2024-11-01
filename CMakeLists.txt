cmake_minimum_required(VERSION 3.16)

#project(spiderOS)
#################################
# project settings
#################################
if(DEFINED PROJECT_NAME)
    # search for config by project name
    set(TARGET_PROJECT_NAME "${PROJECT_NAME}")
else()
    # use the spiderOS project if not defined
    set(TARGET_PROJECT_NAME "spiderOS")
endif()

# load project list
include("CMakeProjectList.cmake")

# collect project names
set(PROJECT_LIST "")

foreach(project_config ${PROJECT_CONFIG_LIST})

    # reset variables
    set(PROJECT_NAME "none")

    include("${project_config}")

    if("${PROJECT_NAME}" STREQUAL "none")
        message(WARNING "invalid config file: ${project_config}")
        continue()
    endif()

    set(PROJECT_LIST ${PROJECT_LIST} ${PROJECT_NAME})
endforeach()

if (DEFINED CMAKE_SCRIPT_MODE_FILE)

    # special action: list all project names and return
    foreach(project_name ${PROJECT_LIST})
        message("PROJECT_NAME=${project_name}")
    endforeach()

    return()

else()

    foreach(project_config ${PROJECT_CONFIG_LIST})

        # unset optional variables
        unset(KERNEL_DIR)
        unset(LINKER_FILE_CPU32)
        unset(USER_POST_BUILD_SCRIPT)
        unset(USER_EMBEDDED_RESOURCE_DIRS)

        include("${project_config}")

        if("${TARGET_PROJECT_NAME}" STREQUAL "spiderOS" OR
                "${TARGET_PROJECT_NAME}" STREQUAL "${PROJECT_NAME}")

            set(TARGET_PROJECT_NAME "${PROJECT_NAME}")
            set(PROJECT_CONFIG_PATH "${project_config}")
            break()

        endif()

    endforeach()

    if(NOT "${TARGET_PROJECT_NAME}" STREQUAL "${PROJECT_NAME}")
        message(FATAL_ERROR "project not found: ${TARGET_PROJECT_NAME}")
    endif()

    set(PROJECT_NAME "${TARGET_PROJECT_NAME}" CACHE STRING "Project name")
    set_property(CACHE PROJECT_NAME PROPERTY STRINGS ${PROJECT_LIST})

endif()