# Copyright (c) 2021-present Sparky Studios. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

cmake_minimum_required(VERSION 3.20)

cmake_policy(SET CMP0048 NEW)

list(APPEND CMAKE_MODULE_PATH "$ENV{AM_SDK_PATH}/cmake" "${AM_SDK_PATH}/cmake")
list(APPEND CMAKE_PREFIX_PATH "$ENV{AM_SDK_PATH}/cmake" "${AM_SDK_PATH}/cmake")

# TODO: Replace this project name with the identifier of your plugin
project(PluginIdentifier)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_DEBUG_POSTFIX "_d")

set(BIN_DIR ${CMAKE_CURRENT_BINARY_DIR})
set(OBJ_DIR ${CMAKE_CURRENT_BINARY_DIR}/obj)

if(APPLE)
    set(CMAKE_MACOSX_RPATH 1)
    list(APPEND CMAKE_INSTALL_RPATH "@executable_path" "@loader_path/../shared/")
elseif(UNIX AND NOT ANDROID)
    if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,-rpath,$ORIGIN/../shared/")
        set(CMAKE_INSTALL_RPATH_USE_LINK_PATH FALSE)
    else()
        list(APPEND CMAKE_INSTALL_RPATH $ORIGIN $ORIGIN/../shared/)
        set(CMAKE_INSTALL_RPATH_USE_LINK_PATH FALSE)
    endif()
endif()

# Setup Plugin dependencies
# ------------------------------------------------------------------------------
# TODO: Initialize your plugin dependecies here if needed. This is usually the set of 
# `find_package` calls to 3rd party libraries.

# Setup Amplitude
# ------------------------------------------------------------------------------
find_package(AmplitudeAudioSDK REQUIRED)

# Setup Plugin
# ------------------------------------------------------------------------------

# Source files
set(PLUGIN_SOURCE
    Plugin.h
    Plugin.cpp
    # TODO: Add your other source files here
)

# Includes
include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${AM_SDK_PATH}/include
    # TODO: Add you other include directories here
)

add_library(${PROJECT_NAME} SHARED ${PLUGIN_SOURCE})
target_compile_definitions(${PROJECT_NAME} PRIVATE AM_BUILDSYSTEM_BUILDING_PLUGIN)

target_link_libraries(${PROJECT_NAME}
    PRIVATE SparkyStudios::Audio::Amplitude::SDK::Shared
    # TODO: Add your other libraries to link here
)

# Install
# ------------------------------------------------------------------------------
set(AM_LIB_DESTINATION "lib/${AM_SDK_PLATFORM}")
set(AM_INC_DESTINATION "include")

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/${AM_LIB_DESTINATION}")

set(CMAKE_INSTALL_PREFIX ${AM_SDK_PATH})

install(
    TARGETS ${PROJECT_NAME}
    ARCHIVE DESTINATION ${AM_LIB_DESTINATION}/plugins
    LIBRARY DESTINATION ${AM_LIB_DESTINATION}/plugins
    RUNTIME DESTINATION ${AM_LIB_DESTINATION}/plugins
)

install(
    DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/
    DESTINATION ${AM_INC_DESTINATION}
    FILES_MATCHING PATTERN "*.h*"
)

export(PACKAGE ${PROJECT_NAME})
