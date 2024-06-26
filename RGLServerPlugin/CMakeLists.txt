cmake_minimum_required(VERSION 3.16)
project(RGLServerPlugin)
set(CMAKE_CXX_STANDARD 20)

find_package(gz-cmake3 REQUIRED)
gz_find_package(gz-plugin2 REQUIRED COMPONENTS register)
set(GZ_PLUGIN_VER ${gz-plugin2_VERSION_MAJOR})

gz_find_package(gz-sim8 REQUIRED)
set(GZ_SIM_VER ${gz-sim8_VERSION_MAJOR})


include_directories(include)

set(RGL_VERSION "0.11.3")
set(RGL_PATH "${CMAKE_CURRENT_SOURCE_DIR}/include/rgl")
set(RGL_SO_PATH "${RGL_PATH}/libRobotecGPULidar.so")
set(RGL_API_PATH "${RGL_PATH}/api/core.h")
set(RGL_COMMIT "e5ea8a1a74c253a304f4d1d4652391bca9d448e6")
set(RGL_SO_URL "https://github.com/RobotecAI/RobotecGPULidar/releases/download/v${RGL_VERSION}/libRobotecGPULidar.so")
set(RGL_API_URL "https://raw.githubusercontent.com/RobotecAI/RobotecGPULidar/${RGL_COMMIT}/include/rgl/api/core.h")

# You have to delete old files manually in order to download
# new ones automatically if a new version is available.
if(NOT EXISTS ${RGL_SO_PATH})
    file(DOWNLOAD ${RGL_SO_URL} ${RGL_SO_PATH})
endif()

if(NOT EXISTS ${RGL_API_PATH})
    file(DOWNLOAD ${RGL_API_URL} ${RGL_API_PATH})
endif()

set(RobotecGPULidar ${RGL_SO_PATH})

add_library(RGLServerPluginManager SHARED src/RGLServerPluginManager.cc src/Mesh.cc src/Utils.cc src/Scene.cc)
target_link_libraries(RGLServerPluginManager
    PRIVATE gz-plugin${GZ_PLUGIN_VER}::gz-plugin${GZ_PLUGIN_VER}
    PRIVATE gz-sim${GZ_SIM_VER}::gz-sim${GZ_SIM_VER}
    ${RobotecGPULidar}
)
set_target_properties(RGLServerPluginManager PROPERTIES INSTALL_RPATH "$ORIGIN")

add_library(RGLServerPluginInstance SHARED src/RGLServerPluginInstance.cc src/Lidar.cc src/Utils.cc src/LidarPatternLoader.cc)
target_link_libraries(RGLServerPluginInstance
    PRIVATE gz-plugin${GZ_PLUGIN_VER}::gz-plugin${GZ_PLUGIN_VER}
    PRIVATE gz-sim${GZ_SIM_VER}::gz-sim${GZ_SIM_VER}
    ${RobotecGPULidar}
)
set_target_properties(RGLServerPluginInstance PROPERTIES INSTALL_RPATH "$ORIGIN")

## Install libraries

install(TARGETS RGLServerPluginInstance RGLServerPluginManager
    DESTINATION RGLServerPlugin
)
install(FILES ${RobotecGPULidar}
    DESTINATION RGLServerPlugin
)
