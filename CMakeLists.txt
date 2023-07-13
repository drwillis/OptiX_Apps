cmake_minimum_required(VERSION 3.17)

project( optix_apps )

SET(ENV{OPTIX74_PATH} /usr/local/NVIDIA-OptiX-SDK-7.4.0-linux64-x86_64)
#SET(CUDA_TOOLKIT_ROOT_DIR /usr/local/cuda)
# for Netbeans to recognize the ELF binary
#SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fPIC")
#SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fPIC")
SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -no-pie -fPIC")
SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -no-pie -fPIC")

#SET(CMAKE_BUILD_TYPE "Release")
SET(CMAKE_BUILD_TYPE "Debug")
SET(CMAKE_VERBOSE_MAKEFILE ON)
SET(BUILD_MDL_RENDERER OFF)

set(LOCAL_3RDPARTY "${CMAKE_CURRENT_SOURCE_DIR}/3rdparty")
message("LOCAL_3RDPARTY = " "${LOCAL_3RDPARTY}")

set(CMAKE_MODULE_PATH "${LOCAL_3RDPARTY}/CMake")
message("CMAKE_MODULE_PATH = " "${CMAKE_MODULE_PATH}")

# This contains a macro which generates custom build rules for compiling *.cu input files to either *.ptx or *.optixir.
include("nvcuda_compile_module")
# Use the NVCUDA_COMPILE_PTX function to produce the desired custom rule and output filenames when compiling OptiX programs from *.cu to *.ptx.
include("nvcuda_compile_ptx")

# amd64 is the only supported platform here. # DAR FIXME Remove all other remaining cases.
set(LOCAL_ARCH "amd64")

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

if(UNIX)
  set(OS "linux")
  add_definitions("-DLINUX")
  add_definitions("-Wno-unused-local-typedefs -Wno-delete-non-virtual-dtor")
  set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -static-libstdc++ -static-libgcc")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread")
else(UNIX)
  if(APPLE)
  else(APPLE)
    if(WIN32)
      set(OS "win")
      add_definitions("-DNOMINMAX")
    endif(WIN32)
  endif(APPLE)
endif(UNIX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/lib")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/bin")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_BINARY_DIR}/bin")

#if(WIN32 AND "${CMAKE_GENERATOR}" MATCHES "^(Visual Studio).*")
#  # Set the base folder where the per-project "core" folders with the *.ptx or *.optixir files get created.
#  set(MODULE_TARGET_DIR "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/$(ConfigurationName)")
#  # Enable multi-processor build on all Visual Studio versions.
#  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")
#else()
#  # DAR This should be independent of ${CMAKE_BUILD_TYPE} because that single-configuration generator will not create subfolders, will it?
#  # Otherwise add something with if("${CMAKE_BUILD_TYPE}" STREQUAL "")
#  set(MODULE_TARGET_DIR "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}")
#endif()
if(WIN32 AND "${CMAKE_GENERATOR}" MATCHES "^(Visual Studio).*")
  # Set the base folder where the per-project ptx folders get created.
  set(PTX_TARGET_DIR "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}/$(ConfigurationName)")
  # Enable multi-processor build on all Visual Studio versions.
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")
else()
  # DAR This should be independent of ${CMAKE_BUILD_TYPE} because that single-configuration generator will not create subfolders, will it?
  # Otherwise add something with if("${CMAKE_BUILD_TYPE}" STREQUAL "")
  set(PTX_TARGET_DIR "${CMAKE_RUNTIME_OUTPUT_DIRECTORY}")
endif()


# Some useful macros
macro(ADD_TARGET_PROPERTIES _target _name)
  set(_properties)
  foreach(_prop ${ARGN})
    set(_properties "${_properties} ${_prop}")
  endforeach(_prop)
  get_target_property(_old_properties ${_target} ${_name})
  if(NOT _old_properties)
    # In case it's NOTFOUND
    set(_old_properties)
  endif(NOT _old_properties)
  set_target_properties(${_target} PROPERTIES ${_name} "${_old_properties} ${_properties}")
endmacro(ADD_TARGET_PROPERTIES)

macro(TARGET_INCLUDE_SYMBOL target symbol)
  if (WIN32)
    if ( LOCAL_ARCH STREQUAL "amd64" )
      add_target_properties( ${target} LINK_FLAGS /include:${symbol} )
    endif()
  endif()
  if(UNIX)
    add_target_properties( ${target} LINK_FLAGS "-Wl,--undefined=${symbol}" )
  endif()
endmacro()

# Search for all shipping OptiX 7 versions before adding subdirectories.
# The resp. FindOptiX7*.cmake looks for OPTIX7*_PATH environment variables.
# Doing this here allows excluding examples from the build which require a specific OptiX 7 minor version.
# If multiple OptiX SDK versions are found, the newest is picked by the individual examples.
find_package(OptiX70)
find_package(OptiX71)
find_package(OptiX72)
find_package(OptiX73)
find_package(OptiX74)
find_package(OptiX75)
find_package(OptiX76)
find_package(OptiX77)
# The rtigo9_omm requires the CUDA Opacity Micromap Baking tool inside the OptiX Toolkit.
# The OptiX Toolkit is only built for OptiX SDK 7.6.0 and newer.
find_package(OptiXToolkit)
# The MDL_renderer requires the MDL SDK to generate shader code and load images.
find_package(MDL_SDK)

SET(OpenGL_GL_PREFERENCE LEGACY)
SET(BUILD_WITH_GUI ON)

add_subdirectory(third_party)

SET(GLFW_INCLUDE_DIR third_party/glfw/include)
SET(GLFW_LIBRARIES glfw libglew_shared imgui)
#SET(ILUT_LIBRARIES ILUT)
SET(ILUT_LIBRARIES "")
##SET(ILU_LIBRARIES ILU)
SET(ILU_LIBRARIES "")
SET(IL_LIBRARIES IL)
SET(IL_INCLUDE_DIR third_party/DevIL/DevIL/include)
SET(ASSIMP_LIBRARIES assimp)
SET(ASSIMP_INCLUDE_DIRS third_party/assimp/include ${CMAKE_BINARY_DIR}/third_party/assimp/include)
include_directories(
        third_party/
        third_party/DevIL/DevIL/include
        third_party/imgui
        third_party/glew-cmake/include
        third_party/glfw/include
        ${OPTIX_INCLUDE_DIR}
        ${CUDA_TOOLKIT_INCLUDE}
        #${IL_INCLUDE_DIR}
        third_party/assimp/include
        #${ASSIMP_INCLUDE_DIRS}
        )

add_subdirectory( apps )
