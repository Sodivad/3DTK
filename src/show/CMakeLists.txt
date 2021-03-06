find_package(PNG REQUIRED)
include_directories(${PNG_INCLUDE_DIRS})
add_definitions(${PNG_DEFINITIONS})
add_definitions(-DGLUI_NO_LIB_PRAGMA)

set(SHOW_LIBS ${PNG_LIBRARIES} ${ANN_LIBRARIES_STATIC} ${NEWMAT_LIBRARIES_STATIC} ${Boost_SYSTEM_LIBRARY} ${Boost_FILESYSTEM_LIBRARY} ${Boost_PROGRAM_OPTIONS_LIBRARY})
set(SHOW_LIBS_STATIC scan_static ${SHOW_LIBS} CACHE INTERNAL "")
set(SHOW_LIBS_SHARED scan_shared ${SHOW_LIBS} CACHE INTERNAL "")

if (WITH_OPENGL)
  find_package(OpenGL REQUIRED)
  find_package(GLUT REQUIRED)
  add_definitions(-DWITH_OPENGL)
  if(APPLE)
    include_directories(/usr/local/Cellar/glui/2.36/)
    set(SHOW_LIBS_STATIC ${SHOW_LIBS_STATIC} ${GLUT_LIBRARIES} ${OPENGL_LIBRARIES} /usr/local/Cellar/glui/2.36/lib/libglui.a CACHE INTERNAL "")
    set(SHOW_LIBS_SHARED ${SHOW_LIBS_SHARED} ${GLUT_LIBRARIES} ${OPENGL_LIBRARIES} /usr/local/Cellar/glui/2.36/lib/libglui.a CACHE INTERNAL "")
  else()
    include_directories(${PROJECT_SOURCE_DIR}/3rdparty/glui-2.36/include/)
    set(SHOW_LIBS_STATIC ${SHOW_LIBS_STATIC} ${GLUT_LIBRARIES} ${OPENGL_LIBRARIES} glui_static CACHE INTERNAL "")
    set(SHOW_LIBS_SHARED ${SHOW_LIBS_SHARED} ${GLUT_LIBRARIES} ${OPENGL_LIBRARIES} glui_shared CACHE INTERNAL "")
  endif()
  if(WITH_GLEE)
    message(STATUS "Using opengl extensions")
    set(SHOW_LIBS_STATIC ${SHOW_LIBS_STATIC} glee_static CACHE INTERNAL "")
    set(SHOW_LIBS_SHARED ${SHOW_LIBS_SHARED} glee_shared CACHE INTERNAL "")
    add_definitions(-DWITH_GLEE)
  else()
    message(STATUS "Not using opengl extensions")
  endif()
endif()

add_library(show_objects OBJECT NurbsPath.cc PathGraph.cc scancolormanager.cc colormanager.cc compacttree.cc show_gl.cc vertexarray.cc viewcull.cc display.cc show_animate.cc show_common.cc show_menu.cc program_options.cc callbacks_glut.cpp)

# compile with -fPIC so that the result can be made into a shared library
set_property(TARGET show_objects PROPERTY POSITION_INDEPENDENT_CODE 1)

add_library(show_static STATIC show.cc $<TARGET_OBJECTS:show_objects>)
add_library(show_shared SHARED show.cc $<TARGET_OBJECTS:show_objects>)

set_target_properties(show_static PROPERTIES OUTPUT_NAME show CLEAN_DIRECT_OUTPUT 1)
set_target_properties(show_shared PROPERTIES OUTPUT_NAME show CLEAN_DIRECT_OUTPUT 1)

target_link_libraries(show_static ${SHOW_LIBS_STATIC})
target_link_libraries(show_shared ${SHOW_LIBS_SHARED})

# FIXME: is opengl necessary for everything including the libs?
if (WITH_OPENGL)
  include_directories(${GLUT_INCLUDE_DIR})

  if(WITH_COMPACT_OCTREE)
    message(STATUS "Using compact octrees")
    add_definitions(-DUSE_COMPACT_TREE)
  else()
    message(STATUS "Not using compact octreees: Warning uses more memory")
  endif()

  add_executable(show show.cc $<TARGET_OBJECTS:show_objects>)
  target_link_libraries(show ${SHOW_LIBS_SHARED})

  if(WITH_WXWIDGETS)
    find_package(wxWidgets REQUIRED core base gl)
    # set wxWidgets_wxrc_EXECUTABLE to be ignored in the configuration
    set (wxWidgets_wxrc_EXECUTABLE " " CACHE INTERNAL "" FORCE)
    # wxWidgets include (this will do all the magic to configure everything)
    include( ${wxWidgets_USE_FILE})
    add_executable(wxshow wxshow.cc selectionframe.cc $<TARGET_OBJECTS:show_objects>)
    target_link_libraries(wxshow ${wxWidgets_LIBRARIES} wxthings ${SHOW_LIBS_SHARED})
  else()
    message(STATUS "Not building wxshow because WITH_WXWIDGETS is OFF")
  endif()
else()
  message(STATUS "Not building show because WITH_OPENGL is OFF")
endif()
