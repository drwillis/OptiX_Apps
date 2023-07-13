#
# Copyright (C) 2021 Andrew R. Willis
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.
#

cmake_minimum_required(VERSION 3.17 FATAL_ERROR)

##################################################################
# device_include processing
set(CMAKE_POLICY_DEFAULT_CMP0077 NEW)
set(ASSIMP_BUILD_ASSIMP_TOOLS OFF)
set(ASSIMP_BUILD_TESTS OFF)
#set(ASSIMP_INSTALL OFF)
add_subdirectory(assimp)

##################################################################
# DevIL compilation
set (CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/DevIL/DevIL/cmake/Modules")

#set( IL_USE_DXTC_SQUISH OFF)
#set( IL_NO_JP2 OFF)
add_subdirectory( DevIL/DevIL/src-IL )
add_subdirectory( DevIL/DevIL/src-ILU )
#add_subdirectory( DevIL/DevIL/src-ILUT )
set_target_properties(IL PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${CMAKE_LIBRARY_OUTPUT_DIRECTORY})
#set_target_properties(IL PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${CMAKE_LIBRARY_OUTPUT_DIRECTORY})
#set_target_properties(ILU PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${CMAKE_LIBRARY_OUTPUT_DIRECTORY})
#set_target_properties(ILUT PROPERTIES LIBRARY_OUTPUT_DIRECTORY ${CMAKE_LIBRARY_OUTPUT_DIRECTORY})

#file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/../include)
#file(MAKE_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/../include/IL)
#file(GLOB DEVIL_HEADERS
#  "DevIL/DevIL/include/IL/*.h"
#  "DevIL/DevIL/include/IL/il.h"
#)
#file(COPY ${DEVIL_HEADERS} DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/../include/IL)

if (BUILD_WITH_GUI)
    ##################################################################
    # GLFW compilation

    set(GLFW_LIBRARY_TYPE SHARED)
    set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
    set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
    set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)
    add_subdirectory( glfw )

    ##################################################################
    # IMGUI compilation

    include_directories(imgui)

    add_library( imgui SHARED
        imgui/imgui.cpp
        imgui/imgui_demo.cpp
        imgui/imgui_draw.cpp    
        imgui/imconfig.h
        imgui/imgui.h
        imgui/imgui_tables.cpp
        imgui/imgui_widgets.cpp
        imgui/imgui_internal.h
        imgui/imstb_rectpack.h
        imgui/imstb_textedit.h
        imgui/imstb_truetype.h
        imgui/backends/imgui_impl_glfw.h
        imgui/backends/imgui_impl_glfw.cpp
        imgui/backends/imgui_impl_opengl2.h
        imgui/backends/imgui_impl_opengl2.cpp
        imgui/backends/imgui_impl_opengl3.h
        imgui/backends/imgui_impl_opengl3.cpp
    )

    target_link_libraries( imgui PRIVATE glfw ${OPENGL_gl_LIBRARY})
endif (BUILD_WITH_GUI)
##################################################################
# sutil compilation

# glew always appends "d" for libraries built in debug Release which creates build problems
#SET(CMAKE_BUILD_TYPE_TEMP ${CMAKE_BUILD_TYPE})
#SET(CMAKE_BUILD_TYPE "Release")
#set(ONLY_LIBS ON)
add_subdirectory(glew-cmake)
#SET(CMAKE_BUILD_TYPE ${CMAKE_BUILD_TYPE_TEMP})

#add_subdirectory(glew-2.2.0/build/cmake)
#include_directories(glew-2.2.0/include)
##################################################################
# yaml-cpp compilation

#set(yaml-cpp-type STATIC IMPORTED)
#add_subdirectory(yaml-cpp)
