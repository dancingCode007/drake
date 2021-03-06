set(drakeCollision_SRC_FILES drake_collision.cc element.cc model.cc)

if(Bullet_FOUND)
  set(drakeCollision_SRC_FILES ${drakeCollision_SRC_FILES} bullet_model.cc)
  set(bullet "yes" CACHE STRING "yes")
endif()

if(ccd_FOUND AND fcl_FOUND AND octomap_FOUND)
  list(APPEND drakeCollision_SRC_FILES fcl_model.cc)
endif()

set(drakeCollision_SRC_FILES ${drakeCollision_SRC_FILES} unusable_model.cc)

add_library_with_exports(LIB_NAME drakeCollision SOURCE_FILES ${drakeCollision_SRC_FILES})

set(bullet_dependent_requires)
if(Bullet_FOUND)
  target_compile_definitions(drakeCollision PUBLIC BULLET_COLLISION)
  target_include_directories(drakeCollision PUBLIC ${BULLET_INCLUDE_DIRS})
  target_link_libraries(drakeCollision ${BULLET_LIBRARIES})
  set(bullet_dependent_requires bullet)
endif()

set(fcl_dependent_requires)
if(ccd_FOUND AND fcl_FOUND AND octomap_FOUND)
  target_link_libraries(drakeCollision ccd fcl octomap)
  set(fcl_dependent_requires ccd fcl octomap)
endif()

target_link_libraries(drakeCollision
  drakeCommon
  drakeShapes
  Eigen3::Eigen)

drake_install_libraries(drakeCollision)
drake_install_headers(drake_collision.h point_pair.h element.h model.h)
drake_install_pkg_config_file(drake-collision
  TARGET drakeCollision
  LIBS -ldrakeCollision
  REQUIRES
    drake-common
    drake-shapes
    eigen3
    ${bullet_dependent_requires}
    ${fcl_dependent_requires})

if(BUILD_TESTING)
  add_subdirectory(test)
endif()
