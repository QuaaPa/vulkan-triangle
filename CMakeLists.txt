# ----------------------------------------
#
# release configurations with this command:
#     cmake -B build . -DCMAKE_BUILD_TYPE=Release
#
# ninja generator:
#     cmake -G Ninja -B build . -DCMAKE_BUILD_TYPE=Release
#
# ----------------------------------------
# ----------------------------------------
#
# debug configurations with this command:
#     cmake -B build . -DCMAKE_BUILD_TYPE=Debug
#
# ninja generator:
#     cmake -G Ninja -B build . -DCMAKE_BUILD_TYPE=Debug
#
# ----------------------------------------

cmake_minimum_required(VERSION 3.10)

set(PROJECT_NAME VulkanAPP)
project(${PROJECT_NAME})

set(CMAKE_CXX_STANDARD 17)

# CONFIGURE_DEPENDS ---------------------------------------------------		       
# Tell CMake about additional input files to the configuration process.
# If any named file is modified the build system will re-run CMake
# to re-configure the file and generate the build system again.
# ---------------------------------------------------------------------

file(GLOB_RECURSE SOURCES CONFIGURE_DEPENDS "src/*.cc")
file(GLOB_RECURSE HEADERS "src/*.h" "src/*.hh")

find_package(Vulkan REQUIRED)
find_package(glfw3 CONFIG REQUIRED)

add_executable(${PROJECT_NAME} ${SOURCES} ${HEADERS})

target_link_libraries(${PROJECT_NAME} Vulkan::Vulkan glfw)


set_target_properties(${PROJECT_NAME} PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin/)

set(SPV_OUTPUTS "")

file(GLOB SHADER_SOURCES "res/shaders/*.vert" "res/shaders/*.frag")
foreach(source IN LISTS SHADER_SOURCES)
  get_filename_component(OUTPUT_NAME ${source} NAME_WE)
  set(SPV_OUT ${CMAKE_BINARY_DIR}/bin/res/shaders/${OUTPUT_NAME})
  add_custom_command(
    OUTPUT ${SPV_OUT}
    COMMAND glslc ${source} -o ${SPV_OUT}.spv
    DEPENDS ${source}
  )
  list(APPEND SPV_OUTPUTS ${SPV_OUT})
endforeach()

add_custom_target(Shaders ALL DEPENDS ${SPV_OUTPUTS})
add_dependencies(${PROJECT_NAME} Shaders)

add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory ${CMAKE_SOURCE_DIR}/res $<TARGET_FILE_DIR:${PROJECT_NAME}>/res
)
