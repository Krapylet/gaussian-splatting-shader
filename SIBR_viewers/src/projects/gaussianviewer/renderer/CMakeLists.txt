# Copyright (C) 2023, Inria
# GRAPHDECO research group, https://team.inria.fr/graphdeco
# All rights reserved.
# 
# This software is free for non-commercial, research and evaluation use 
# under the terms of the LICENSE.md file.
# 
# For inquiries contact sibr@inria.fr and/or George.Drettakis@inria.fr

set(SIBR_PROJECT "gaussian")
project(sibr_${SIBR_PROJECT} LANGUAGES CXX)

#sibr_gitlibrary(TARGET CudaRasterizer
#    GIT_REPOSITORY 	"https://github.com/graphdeco-inria/diff-gaussian-rasterization.git"
#    GIT_TAG			"3509be80f83ee30599b23bb3542d45aea2174a03"
#    GIT_REPOSITORY 	"https://github.com/Krapylet/diff-gaussian-rasterization.git"
#    GIT_TAG			"origin/main"
#)

### <Manually loading local razterizer lib.>
set(args_TARGET CudaRasterizer)
set(args_ROOT_DIR ${args_TARGET})
set(args_SOURCE_DIR ${args_TARGET})

set(LOCAL_RASTERIZER_PATH "C:/Users/asvj/Documents/GaussianSplatRenderer/diff-gaussian-rasterization")
FetchContent_Declare(${args_TARGET}
	SOURCE_DIR 		${LOCAL_RASTERIZER_PATH}
	SUBBUILD_DIR    ${CMAKE_SOURCE_DIR}/extlibs/CudaRasterizer/subbuild
	BINARY_DIR      ${CMAKE_SOURCE_DIR}/extlibs/CudaRasterizer/build
)

FetchContent_GetProperties(${args_TARGET})
string(TOLOWER "<name>" lcTargetName)

if(NOT ${lcTargetName}_POPULATED)
	message(STATUS "Populating library ${args_TARGET}...")
	FetchContent_Populate(${args_TARGET}
		SOURCE_DIR 		${LOCAL_RASTERIZER_PATH}
		SUBBUILD_DIR    ${CMAKE_SOURCE_DIR}/extlibs/${args_ROOT_DIR}/subbuild
		BINARY_DIR      ${CMAKE_SOURCE_DIR}/extlibs/${args_ROOT_DIR}/build
	)
endif()

add_subdirectory(${LOCAL_RASTERIZER_PATH} ${CMAKE_SOURCE_DIR}/extlibs/${args_ROOT_DIR}/build)

get_target_property(type ${args_TARGET} TYPE)
if(NOT (type STREQUAL "INTERFACE_LIBRARY"))
	set_target_properties(${args_TARGET} PROPERTIES FOLDER "extlibs")
endif()
		
list(APPEND ${args_TARGET}_INCLUDE_DIRS ${EXTLIBS_PACKAGE_FOLDER}/${args_ROOT_DIR})
list(APPEND ${args_TARGET}_INCLUDE_DIRS ${LOCAL_RASTERIZER_PATH})

foreach(args_INCLUDE_DIR ${args_INCLUDE_DIRS})
	list(APPEND ${args_TARGET}_INCLUDE_DIRS ${EXTLIBS_PACKAGE_FOLDER}/${args_ROOT_DIR}/${args_SOURCE_DIR}/${args_INCLUDE_DIR})
endforeach()

include_directories(${${args_TARGET}_INCLUDE_DIRS})

### </Manually loading local razterizer lib.>




find_package(CUDAToolkit REQUIRED)

file(GLOB SOURCES "*.cpp" "*.h" "*.hpp")
source_group("Source Files" FILES ${SOURCES})

file(GLOB SHADERS "shaders/*.frag" "shaders/*.vert" "shaders/*.geom")
source_group("Source Files\\shaders" FILES ${SHADERS})

file(GLOB SOURCES "*.cpp" "*.h" "*.hpp" "shaders/*.frag" "shaders/*.vert" "shaders/*.geom")

## Specify target rules
add_library(${PROJECT_NAME} SHARED ${SOURCES})

include_directories(${Boost_INCLUDE_DIRS} .)
if (WIN32)
target_link_libraries(${PROJECT_NAME}
	${Boost_LIBRARIES}
	${ASSIMP_LIBRARIES}
	${GLEW_LIBRARIES}
	${OPENGL_LIBRARIES}
	${OpenCV_LIBRARIES}
	glfw3
	sibr_system
	sibr_view
	sibr_assets
	sibr_renderer
	sibr_basic
	CUDA::cudart
	CudaRasterizer
)
else()
target_link_libraries(${PROJECT_NAME}
	${Boost_LIBRARIES}
	${ASSIMP_LIBRARIES}
	${GLEW_LIBRARIES}
	${OPENGL_LIBRARIES}
	${OpenCV_LIBRARIES}
	glfw
	sibr_system
	sibr_view
	sibr_assets
	sibr_renderer
	sibr_basic
	CUDA::cudart
	CudaRasterizer
)
endif()

add_definitions( -DSIBR_EXP_ULR_EXPORTS -DBOOST_ALL_DYN_LINK  )

set_target_properties(${PROJECT_NAME} PROPERTIES FOLDER "projects/${SIBR_PROJECT}/renderer")

## High level macro to install in an homogen way all our ibr targets
include(install_runtime)
ibr_install_target(${PROJECT_NAME}
    INSTALL_PDB                         ## mean install also MSVC IDE *.pdb file (DEST according to target type)
	SHADERS ${SHADERS}
	RSC_FOLDER ${SIBR_PROJECT}

    #STANDALONE  ${INSTALL_STANDALONE}   ## mean call install_runtime with bundle dependencies resolution
    COMPONENT   ${PROJECT_NAME}_install ## will create custom target to install only this project
)
