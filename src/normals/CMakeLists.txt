if (0)
  add_definitions(-DWITH_OPENCV)
  include_directories(${NEWMAT_INCLUDE_DIRS})

  add_library(normals_panorama_static STATIC normals_panorama.cc)
  add_library(normals_panorama_shared SHARED normals_panorama.cc)

  set_target_properties(normals_panorama_static PROPERTIES OUTPUT_NAME normals_panorama CLEAN_DIRECT_OUTPUT 1)
  set_target_properties(normals_panorama_shared PROPERTIES OUTPUT_NAME normals_panorama CLEAN_DIRECT_OUTPUT 1)

  target_link_libraries(normals_panorama_static ${NEWMAT_LIBRARIES_STATIC} scan_static fbr_cv_io_static fbr_panorama_static ${OpenCV_LIBS})
  target_link_libraries(normals_panorama_shared ${NEWMAT_LIBRARIES_SHARED} scan_shared fbr_shared ${OpenCV_LIBS} ${Boost_LIBRARIES})

  set(NORMALS_PANORAMA_LIBRARIES normals_panorama_static)

  add_executable(calc_normals calc_normals.cc)
  target_link_libraries(calc_normals scan_static normals_panorama_static ${Boost_LIBRARIES} ${Boost_SYSTEM_LIBRARY} ${Boost_FILESYSTEM_LIBRARY} ${SHOW_LIBS_STATIC})
endif()