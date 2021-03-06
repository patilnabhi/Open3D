# some black magic to automatically create a header file from shader files
file(GLOB SHADER_FILES "Shader/GLSL/*.glsl")

add_custom_command(
    OUTPUT ${PROJECT_SOURCE_DIR}/src/Open3D/Visualization/Shader/Shader.h
    COMMAND EncodeShader Shader.h
    WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/src/Open3D/Visualization/Shader/
    DEPENDS ${SHADER_FILES} EncodeShader
)

foreach(file ${SHADER_FILES})
    add_custom_command(
        OUTPUT ${PROJECT_SOURCE_DIR}/src/Open3D/Visualization/Shader/Shader.h
        COMMAND EncodeShader Shader.h ${file}
        WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/src/Open3D/Visualization/Shader/
        DEPENDS ${SHADER_FILES} EncodeShader
        APPEND
    )
endforeach(file)

add_custom_target(ShaderFileTarget ALL
                  DEPENDS ${PROJECT_SOURCE_DIR}/src/Open3D/Visualization/Shader/Shader.h)
set_target_properties(ShaderFileTarget PROPERTIES FOLDER "Custom")

# build
set(VISUALIZATION_ALL_HEADER_FILES "")
set(VISUALIZATION_ALL_SOURCE_FILES "")
macro(AddSourceDir DIR)
    file(GLOB_RECURSE HEADER_FILES "${DIR}/*.h")
    file(GLOB_RECURSE SOURCE_FILES "${DIR}/*.cpp")
    list(APPEND VISUALIZATION_ALL_HEADER_FILES ${HEADER_FILES})
    list(APPEND VISUALIZATION_ALL_SOURCE_FILES ${SOURCE_FILES})
endmacro(AddSourceDir)
AddSourceDir("Shader")
AddSourceDir("Utility")
AddSourceDir("Visualizer")

if (ENABLE_GUI)
    AddSourceDir("Rendering")
else ()
    list(FILTER VISUALIZATION_ALL_HEADER_FILES EXCLUDE REGEX .*GuiVisualizer.h)
    list(FILTER VISUALIZATION_ALL_SOURCE_FILES EXCLUDE REGEX .*GuiVisualizer.cpp)
endif()

# create object library
add_library(Visualization OBJECT
            ${SHADER_FILES}
            ${VISUALIZATION_ALL_HEADER_FILES}
            ${VISUALIZATION_ALL_SOURCE_FILES})

source_group("Source Files\\Shader\\GLSL" FILES ${SHADER_FILES})

add_dependencies(Visualization ShaderFileTarget)

open3d_show_and_abort_on_warning(Visualization)
open3d_set_global_properties(Visualization)
open3d_link_3rdparty_libraries(Visualization)
