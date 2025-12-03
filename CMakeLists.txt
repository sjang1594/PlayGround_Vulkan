cmake_minimum_required(VERSION 3.21)

project(PlayGround_Vulkan VERSION 1.0.0 LANGUAGES CXX)

# Set C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Export compile commands for IDE support
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Find Vulkan (installed via vcpkg or system)
find_package(Vulkan REQUIRED)
message(STATUS "Vulkan found: ${Vulkan_LIBRARY}")

# Find GLFW for window management (installed via vcpkg)
find_package(glfw3 CONFIG REQUIRED)
message(STATUS "GLFW3 found")

# Find GLM for math (installed via vcpkg)
find_package(glm CONFIG REQUIRED)
message(STATUS "GLM found")

# Find SPIRV-Reflect for shader reflection (installed via vcpkg)
find_package(unofficial-spirv-reflect CONFIG REQUIRED)
message(STATUS "SPIRV-Reflect found")

# Find ImGui (installed via vcpkg)
find_package(imgui CONFIG REQUIRED)
message(STATUS "ImGui found")

# Find STB (header-only library)
find_package(Stb REQUIRED)
message(STATUS "STB found")

# Collect source files (when they exist)
file(GLOB_RECURSE SOURCES
    "${CMAKE_SOURCE_DIR}/src/*.cpp"
    "${CMAKE_SOURCE_DIR}/src/*.c"
)

file(GLOB_RECURSE HEADERS
    "${CMAKE_SOURCE_DIR}/src/*.h"
    "${CMAKE_SOURCE_DIR}/src/*.hpp"
)

# Create executable
if(SOURCES)
    add_executable(${PROJECT_NAME} ${SOURCES} ${HEADERS})

    # Link libraries
    target_link_libraries(${PROJECT_NAME} PRIVATE
        Vulkan::Vulkan
        glfw
        glm::glm
        unofficial::spirv-reflect
        imgui::imgui
    )

    # Include directories
    target_include_directories(${PROJECT_NAME} PRIVATE
        ${CMAKE_SOURCE_DIR}/src
        ${Vulkan_INCLUDE_DIRS}
        ${Stb_INCLUDE_DIR}
    )

    # Compiler-specific flags
    if(MSVC)
        target_compile_options(${PROJECT_NAME} PRIVATE
            /W4        # Warning level 4
            /WX-       # Warnings not as errors (remove - to make warnings errors)
        )
    else()
        target_compile_options(${PROJECT_NAME} PRIVATE
            -Wall
            -Wextra
            -Wpedantic
        )
    endif()

    # Set output directories
    set_target_properties(${PROJECT_NAME} PROPERTIES
        RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
    )

else()
    message(WARNING "No source files found in src/ directory. Add .cpp files to build the project.")
endif()

# Optional: Add shader compilation support
# Uncomment when you have shaders to compile
# find_program(GLSL_VALIDATOR glslangValidator HINTS ${Vulkan_GLSLANG_VALIDATOR_EXECUTABLE})
#
# if(GLSL_VALIDATOR)
#     file(GLOB_RECURSE GLSL_SOURCE_FILES
#         "${CMAKE_SOURCE_DIR}/shaders/*.frag"
#         "${CMAKE_SOURCE_DIR}/shaders/*.vert"
#         "${CMAKE_SOURCE_DIR}/shaders/*.comp"
#     )
#
#     foreach(GLSL ${GLSL_SOURCE_FILES})
#         get_filename_component(FILE_NAME ${GLSL} NAME)
#         set(SPIRV "${CMAKE_BINARY_DIR}/shaders/${FILE_NAME}.spv")
#         add_custom_command(
#             OUTPUT ${SPIRV}
#             COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_BINARY_DIR}/shaders/"
#             COMMAND ${GLSL_VALIDATOR} -V ${GLSL} -o ${SPIRV}
#             DEPENDS ${GLSL}
#         )
#         list(APPEND SPIRV_BINARY_FILES ${SPIRV})
#     endforeach()
#
#     add_custom_target(Shaders DEPENDS ${SPIRV_BINARY_FILES})
#     add_dependencies(${PROJECT_NAME} Shaders)
# endif()