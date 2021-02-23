cmake_minimum_required(VERSION 3.1.0)

if (NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "No build type selected, default to Debug")
    set(CMAKE_BUILD_TYPE "Debug")
endif()

project(chronotools VERSION 0.1 LANGUAGES C CXX)

if(WIN32 OR "${CMAKE_SYSTEM}" MATCHES "Windows")
    option(USE_ICU "Use ICU library instead of Win32 API for codepage conversion." OFF)
else()
    set(USE_ICU TRUE)
endif()

option(BUILD_GUI_TOOLS "Build GUI versions of tools where available (Requires wxWidgets)." OFF)

if(USE_ICU)
    find_package(ICU REQUIRED data i18n io tu uc)
endif()

if(BUILD_GUI_TOOLS)
    find_package(wxWidgets 3.0.4 COMPONENTS core base xrc xml adv)

    if(NOT wxWidgets_FOUND)
        # We are going to use the FetchContent module to grab crashpad and its dependencies.
        include(FetchContent)

        FetchContent_Declare(
            wxwidgets_git
            GIT_REPOSITORY https://github.com/wxWidgets/wxWidgets.git
            GIT_TAG        v3.1.4
            GIT_PROGRESS   TRUE
            GIT_SHALLOW    TRUE
        )
        
        # Do we want static linking or not?
        set(wxBUILD_SHARED OFF CACHE BOOL "Build wx libraries as shared libs")

        # We don't use FetchContent_MakeAvailable here because we don't want all wxwidgets_git targets including, just our dependency.
        FetchContent_GetProperties(wxwidgets_git)
        if(NOT wxwidgets_git_POPULATED)
            FetchContent_Populate(wxwidgets_git)
            add_subdirectory(${wxwidgets_git_SOURCE_DIR} ${wxwidgets_git_BINARY_DIR} EXCLUDE_FROM_ALL)
        endif()
        set(wxWidgets_LIBRARIES wxcore wxbase wxxrc wxxml wxadv)
        set(wxWidgets_wxrc_EXECUTABLE wxrc)
    endif()
endif()

# Set version info for the base config module
set(GITINFO_VERSION_MAJOR ${PROJECT_VERSION_MAJOR})
set(GITINFO_VERSION_MINOR ${PROJECT_VERSION_MINOR})
set(GITINFO_VERSION_PATCH ${PROJECT_VERSION_PATCH})

# Add base module
add_subdirectory(libs/baseconfig EXCLUDE_FROM_ALL)

# Add logging module
set(CAPTNLOG_LEVEL $<$<CONFIG:Debug>:5>$<$<CONFIG:RelWithDebInfo>:0>$<$<CONFIG:Release>:0>$<$<CONFIG:MinSizeRel>:0>)
set(CAPTNASSERT_LEVEL 0)
add_subdirectory(libs/captainslog EXCLUDE_FROM_ALL)

add_subdirectory(src)
