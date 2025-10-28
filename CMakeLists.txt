cmake_minimum_required(VERSION 4.0)

project(imgui_cmake_wrapper)

set(IMGUI_RENDERER_BACKEND
    "opengl3"
    CACHE
      STRING
      "Allows useres to choose which renderer(e.g. OpenGL2,OpenGL3,Vulkan) they want to build imgui for"
)

set(IMGUI_PLATFORM_BACKEND
    "glfw"
    CACHE
      STRING
      "Allows useres to choose which platform(e.g. glfw,SDL2,SDL3) they want to build imgui for"
)

file(
  DOWNLOAD
  https://github.com/cpm-cmake/CPM.cmake/releases/download/v0.40.8/CPM.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/cmake/CPM.cmake
  EXPECTED_HASH
    SHA256=78ba32abdf798bc616bab7c73aac32a17bbd7b06ad9e26a6add69de8f3ae4791)
include(${CMAKE_CURRENT_BINARY_DIR}/cmake/CPM.cmake)

cpmaddpackage(
  NAME
  imgui
  GITHUB_REPOSITORY
  ocornut/imgui
  VERSION
  1.92.4
  DOWNLOAD_ONLY)

add_library(imgui_cmake_wrapper STATIC)

target_sources(
  imgui_cmake_wrapper
  PRIVATE ${imgui_SOURCE_DIR}/imgui.cpp
          ${imgui_SOURCE_DIR}/imgui_draw.cpp
          ${imgui_SOURCE_DIR}/imgui_demo.cpp
          ${imgui_SOURCE_DIR}/imgui_tables.cpp
          ${imgui_SOURCE_DIR}/imgui_widgets.cpp
          ${imgui_SOURCE_DIR}/misc/cpp/imgui_stdlib.cpp)

if(${IMGUI_RENDERER_BACKEND} STREQUAL "opengl2")
  message(STATUS "Build imgui for OpenGL2")
  target_sources(imgui_cmake_wrapper
                 PRIVATE ${imgui_SOURCE_DIR}/backends/imgui_impl_opengl2.cpp)
  find_package(OpenGL 2 EXACT REQUIRED)
  target_link_libraries(imgui_cmake_wrapper PUBLIC OpenGL::OpenGL)
elseif(${IMGUI_RENDERER_BACKEND} STREQUAL "opengl3")
  message(STATUS "Build imgui for OpenGL3")
  target_sources(imgui_cmake_wrapper
                 PRIVATE ${imgui_SOURCE_DIR}/backends/imgui_impl_opengl3.cpp)
  find_package(OpenGL 3 EXACT REQUIRED)
  target_link_libraries(imgui_cmake_wrapper PUBLIC OpenGL::OpenGL)
elseif(${IMGUI_RENDERER_BACKEND} STREQUAL "vulkan")
  message(STATUS "Build imgui for Vulkan")
  target_sources(imgui_cmake_wrapper
                 PRIVATE ${imgui_SOURCE_DIR}/backends/imgui_impl_vulkan.cpp)
  find_package(Vulkan REQUIRED)
  target_link_libraries(imgui_cmake_wrapper PUBLIC Vulkan::Vulkan)
endif()

if(${IMGUI_PLATFORM_BACKEND} STREQUAL "glfw")
  message(STATUS "Build imgui for glfw")

  cpmaddpackage(
    NAME
    GLFW
    GITHUB_REPOSITORY
    glfw/glfw
    GIT_TAG
    3.4
    OPTIONS
    "GLFW_BUILD_TESTS OFF"
    "GLFW_BUILD_EXAMPLES OFF"
    "GLFW_BUILD_DOCS OFF")

  target_sources(imgui_cmake_wrapper
                 PRIVATE ${imgui_SOURCE_DIR}/backends/imgui_impl_glfw.cpp)

  target_link_libraries(imgui_cmake_wrapper PUBLIC glfw)
elseif(${IMGUI_PLATFORM_BACKEND} STREQUAL "sdl2")
  message(STATUS "Build imgui for SDL2")

  cpmaddpackage(NAME SDL2 GITHUB_REPOSITORY libsdl-org/SDL GIT_TAG
                release-2.32.10)

  target_sources(imgui_cmake_wrapper
                 PRIVATE ${imgui_SOURCE_DIR}/backends/imgui_impl_sdl2.cpp)
  target_link_libraries(imgui_cmake_wrapper PUBLIC SDL2)
elseif(${IMGUI_PLATFORM_BACKEND} STREQUAL "sdl3")
  message(STATUS "Build imgui for SDL3")

  cpmaddpackage(NAME SDL3 GITHUB_REPOSITORY libsdl-org/SDL GIT_TAG
                release-3.2.24)

  target_sources(imgui_cmake_wrapper
                 PRIVATE ${imgui_SOURCE_DIR}/backends/imgui_impl_sdl3.cpp)
  target_link_libraries(imgui_cmake_wrapper PUBLIC SDL3)
endif()
target_include_directories(
  imgui_cmake_wrapper PUBLIC ${imgui_SOURCE_DIR} ${imgui_SOURCE_DIR}/backends
                             ${imgui_SOURCE_DIR}/misc/cpp)
