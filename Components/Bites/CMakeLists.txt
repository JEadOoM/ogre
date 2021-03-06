#-------------------------------------------------------------------
# This file is part of the CMake build system for OGRE
#     (Object-oriented Graphics Rendering Engine)
# For the latest info, see http://www.ogre3d.org/
#
# The contents of this file are placed in the public domain. Feel
# free to make use of it in any way you like.
#-------------------------------------------------------------------

############################################################
# Paging optional component
############################################################

project(OgreBites)

if(SDL2_FOUND)
  include_directories(${SDL2_INCLUDE_DIR})
  set(DEPENDENCIES ${DEPENDENCIES} ${SDL2_LIBRARY})
endif()

if(ANDROID)
    include_directories(${ANDROID_NDK}/sources/android/native_app_glue)
endif()

if(EMSCRIPTEN)
    add_definitions(-s USE_SDL=2)
endif()

if (OGRE_STATIC)
  # Static linking means we need to directly use plugins
  include_directories(${OGRE_SOURCE_DIR}/PlugIns/BSPSceneManager/include)
  include_directories(${OGRE_SOURCE_DIR}/PlugIns/CgProgramManager/include)
  include_directories(${OGRE_SOURCE_DIR}/PlugIns/OctreeSceneManager/include)
  include_directories(${OGRE_SOURCE_DIR}/PlugIns/OctreeZone/include)
  include_directories(${OGRE_SOURCE_DIR}/PlugIns/ParticleFX/include)
  include_directories(${OGRE_SOURCE_DIR}/PlugIns/PCZSceneManager/include)
  include_directories(${OGRE_SOURCE_DIR}/RenderSystems/Direct3D9/include ${DirectX9_INCLUDE_DIR})
  include_directories(${OGRE_SOURCE_DIR}/RenderSystems/Direct3D11/include)
  include_directories(${OGRE_SOURCE_DIR}/RenderSystems/GLES/include)
  include_directories(${OGRE_SOURCE_DIR}/RenderSystems/GLES2/include)
  include_directories(${OGRE_SOURCE_DIR}/RenderSystems/GLSupport/include)
  include_directories(${OGRE_SOURCE_DIR}/RenderSystems/GL/include)
  include_directories(${OGRE_SOURCE_DIR}/RenderSystems/GL3Plus/include)

  # Link to all enabled plugins
  set(OGRE_LIBRARIES ${OGRE_LIBRARIES} ${SAMPLE_DEPENDENCIES})
endif ()

if (OGRE_BUILD_COMPONENT_RTSHADERSYSTEM)
  ogre_add_component_include_dir(RTShaderSystem)
  set(DEPENDENCIES ${DEPENDENCIES} OgreRTShaderSystem)
endif ()

# define header and source files for the library
file(GLOB HEADER_FILES "${CMAKE_CURRENT_SOURCE_DIR}/include/*.h")
list(APPEND HEADER_FILES ${CMAKE_BINARY_DIR}/include/OgreBitesPrerequisites.h)
set(SOURCES 
  "${CMAKE_CURRENT_SOURCE_DIR}/src/OgreAdvancedRenderControls.cpp"
  "${CMAKE_CURRENT_SOURCE_DIR}/src/OgreApplicationContext.cpp"
  "${CMAKE_CURRENT_SOURCE_DIR}/src/OgreBitesConfigDialog.cpp"
  "${CMAKE_CURRENT_SOURCE_DIR}/src/OgreCameraMan.cpp"
  "${CMAKE_CURRENT_SOURCE_DIR}/src/OgreSGTechniqueResolverListener.cpp"
  "${CMAKE_CURRENT_SOURCE_DIR}/src/OgreStaticPluginLoader.cpp"
  "${CMAKE_CURRENT_SOURCE_DIR}/src/OgreTrays.cpp")

if(ANDROID OR EMSCRIPTEN OR APPLE_IOS OR WINDOWS_STORE OR WINDOWS_PHONE)
  # no config dialog available
elseif(WIN32)
  include_directories("${CMAKE_CURRENT_SOURCE_DIR}/src")
  list(APPEND SOURCES 
    "${CMAKE_CURRENT_SOURCE_DIR}/src/OgreWIN32ConfigDialog.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/resource.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/misc/OgreWin32Resources.rc")
  # Need to include resource files so that icons are linked
  set(RESOURCE_FILES
    "${CMAKE_CURRENT_SOURCE_DIR}/src/resource.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/misc/OgreWin32Resources.rc"
  )
  source_group(Resources FILES ${RESOURCE_FILES})
elseif(APPLE)
  list(APPEND SOURCES 
    "${CMAKE_CURRENT_SOURCE_DIR}/src/OgreOSXConfigDialog.mm"
    "${CMAKE_CURRENT_SOURCE_DIR}/misc/ogrelogo.png")
  set(RESOURCE_FILES "${CMAKE_CURRENT_SOURCE_DIR}/misc/ogrelogo.png")
  source_group(Resources FILES ${RESOURCE_FILES})
  set(DEPENDENCIES ${DEPENDENCIES} "-framework Cocoa")
elseif(UNIX)
  list(APPEND SOURCES "${CMAKE_CURRENT_SOURCE_DIR}/src/OgreGLXConfigDialog.cpp")
  install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/misc/GLX_backdrop.png" DESTINATION "share/OGRE/")
endif()

# include headers
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
include_directories(${OGRE_SOURCE_DIR}/OgreMain/include)
ogre_add_component_include_dir(Overlay)

# setup target
ogre_add_library_to_folder(Components OgreBites ${OGRE_COMP_LIB_TYPE} ${HEADER_FILES} ${SOURCES})
set_target_properties(OgreBites PROPERTIES VERSION ${OGRE_SOVERSION} SOVERSION ${OGRE_SOVERSION})
target_link_libraries(OgreBites OgreMain OgreOverlay ${DEPENDENCIES})

generate_export_header(OgreBites 
    EXPORT_MACRO_NAME _OgreBitesExport
    EXPORT_FILE_NAME ${CMAKE_BINARY_DIR}/include/OgreBitesPrerequisites.h)

# install 
ogre_config_framework(OgreBites)
ogre_config_component(OgreBites)

install(FILES ${HEADER_FILES}
  DESTINATION include/OGRE/Bites
)
