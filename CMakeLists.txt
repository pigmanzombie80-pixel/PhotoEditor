cmake_minimum_required(VERSION 3.15)
project(LightPhotoEditor CXX C)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()
set(CMAKE_CXX_FLAGS_RELEASE "-O2")

# Optional: statically link the MSVC runtime so the shipped .exe doesn't need
# the Visual C++ Redistributable installed on the end user's machine.
# Enable with: cmake .. -DSTATIC_RUNTIME=ON  (combine with vcpkg triplet
# x64-windows-static, see README "Publishing / Distribution" section).
option(STATIC_RUNTIME "Statically link the C++ runtime (MSVC /MT)" OFF)
if(STATIC_RUNTIME AND MSVC)
    set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
endif()

find_package(OpenGL REQUIRED)

find_package(glfw3 QUIET)
if(NOT glfw3_FOUND)
    find_library(GLFW_LIB glfw)
    if(NOT GLFW_LIB)
        message(FATAL_ERROR "GLFW3 not found. Install it, e.g. 'sudo apt install libglfw3-dev' on Debian/Ubuntu, 'brew install glfw' on macOS, or vcpkg on Windows.")
    endif()
endif()

set(IMGUI_DIR ${CMAKE_SOURCE_DIR}/external/imgui)
set(STB_DIR ${CMAKE_SOURCE_DIR}/external/stb)
set(TFD_DIR ${CMAKE_SOURCE_DIR}/external/tinyfiledialogs)

add_executable(PhotoEditor
    src/main.cpp
    src/ImageIO.cpp
    src/Editor.cpp
    src/PaintTools.cpp
    src/Preset.cpp
    ${IMGUI_DIR}/imgui.cpp
    ${IMGUI_DIR}/imgui_draw.cpp
    ${IMGUI_DIR}/imgui_tables.cpp
    ${IMGUI_DIR}/imgui_widgets.cpp
    ${IMGUI_DIR}/imgui_demo.cpp
    ${IMGUI_DIR}/backends/imgui_impl_glfw.cpp
    ${IMGUI_DIR}/backends/imgui_impl_opengl3.cpp
    ${TFD_DIR}/tinyfiledialogs.c
)

target_include_directories(PhotoEditor PRIVATE
    src
    ${IMGUI_DIR}
    ${IMGUI_DIR}/backends
    ${STB_DIR}
    ${TFD_DIR}
)

if(glfw3_FOUND)
    target_link_libraries(PhotoEditor PRIVATE glfw OpenGL::GL)
else()
    target_link_libraries(PhotoEditor PRIVATE ${GLFW_LIB} OpenGL::GL)
endif()

if(UNIX AND NOT APPLE)
    target_link_libraries(PhotoEditor PRIVATE dl pthread)
endif()

if(APPLE)
    target_link_libraries(PhotoEditor PRIVATE "-framework Cocoa" "-framework IOKit" "-framework CoreVideo")
endif()

if(WIN32)
    target_link_libraries(PhotoEditor PRIVATE comdlg32 ole32)
    # Build as a GUI app (no attached console window). Since our entry point
    # is a normal int main() rather than WinMain, tell the linker to keep the
    # standard C runtime startup while targeting the GUI subsystem.
    set_target_properties(PhotoEditor PROPERTIES WIN32_EXECUTABLE TRUE)
    target_link_options(PhotoEditor PRIVATE "/ENTRY:mainCRTStartup")
endif()

if(CMAKE_BUILD_TYPE STREQUAL "Release" AND UNIX)
    target_link_options(PhotoEditor PRIVATE -s)
endif()
