cmake_minimum_required(VERSION 3.14)

project(imgui_cmake_wrapper)

file(
  DOWNLOAD
  https://github.com/cpm-cmake/CPM.cmake/releases/download/v0.40.8/CPM.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/cmake/CPM.cmake
  EXPECTED_HASH SHA256=78ba32abdf798bc616bab7c73aac32a17bbd7b06ad9e26a6add69de8f3ae4791
)
include(${CMAKE_CURRENT_BINARY_DIR}/cmake/CPM.cmake)

CPMAddPackage(
        NAME GLFW
        GITHUB_REPOSITORY glfw/glfw
        GIT_TAG 3.4
        OPTIONS
          "GLFW_BUILD_TESTS OFF"
          "GLFW_BUILD_EXAMPLES OFF"
          "GLFW_BUILD_DOCS OFF"
)

CPMAddPackage(
        NAME imgui
        GITHUB_REPOSITORY ocornut/imgui
        VERSION 1.92.4
        DOWNLOAD_ONLY
)

message(STATUS ${imgui_SOURCE_DIR})

add_library(imgui_cmake_wrapper STATIC)

target_sources(imgui_cmake_wrapper PRIVATE 
                                ${imgui_SOURCE_DIR}/imgui.cpp
                                ${imgui_SOURCE_DIR}/imgui_draw.cpp
                                ${imgui_SOURCE_DIR}/imgui_demo.cpp
                                ${imgui_SOURCE_DIR}/imgui_tables.cpp
                                ${imgui_SOURCE_DIR}/imgui_widgets.cpp
                                ${imgui_SOURCE_DIR}/backends/imgui_impl_opengl3.cpp
                                ${imgui_SOURCE_DIR}/backends/imgui_impl_glfw.cpp
                                ${imgui_SOURCE_DIR}/misc/cpp/imgui_stdlib.cpp)

target_include_directories(imgui_cmake_wrapper PUBLIC
                                ${imgui_SOURCE_DIR}
                                ${imgui_SOURCE_DIR}/backends
                                ${imgui_SOURCE_DIR}/misc/cpp)   

find_package(OpenGL REQUIRED)
                                
target_link_libraries(imgui_cmake_wrapper PUBLIC glfw ${OPENGL_LIBRARIES})

add_executable(imgui_cmake_wrapper_example ${imgui_SOURCE_DIR}/examples/example_glfw_opengl3/main.cpp)
target_link_libraries(imgui_cmake_wrapper_example PUBLIC imgui_cmake_wrapper)
