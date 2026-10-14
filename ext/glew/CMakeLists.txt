find_package(GLEW)
if(NOT USE_LIBNX)
  if((NOT APPLE OR USE_SYSTEM_FFMPEG) AND GLEW_FOUND)
    add_library(system_glew INTERFACE)
    add_library(Ext::GLEW ALIAS system_glew)
    target_link_libraries(system_glew INTERFACE GLEW::GLEW)
  else()
    find_package(OpenGL REQUIRED)
    add_library(glew STATIC
      GL/glew.h
      GL/glxew.h
      GL/wglew.h
      glew.c
    )
    add_library(Ext::GLEW ALIAS glew)
    target_link_libraries(glew PUBLIC ${OPENGL_LIBRARIES})
    target_compile_definitions(glew PUBLIC GLEW_STATIC)
    target_include_directories(glew PUBLIC . ${OPENGL_INCLUDE_DIR})
    set_target_properties(glew PROPERTIES
      EXCLUDE_FROM_ALL ON
    )
    # pspREBOOT's Linux frontend gives the libretro core an offscreen EGL context, where GLX
    # lookups find no display; resolve GL through EGL instead.
    if(LIBRETRO AND UNIX AND NOT APPLE)
      target_compile_definitions(glew PRIVATE GLEW_EGL)
      target_link_libraries(glew PUBLIC EGL)
    endif()
  endif()
endif()
