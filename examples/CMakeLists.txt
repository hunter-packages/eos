# The examples need a few additional dependencies (e.g. boost filesystem and OpenCV highgui):
find_package(OpenCV REQUIRED core highgui)

hunter_add_package(Boost COMPONENTS system filesystem program_options)
find_package(Boost 1.50.0 COMPONENTS system filesystem program_options REQUIRED)
message(STATUS "Boost found at ${Boost_INCLUDE_DIRS}")

# Simple model fitting (orthographic camera & shape to landmarks) example:
add_executable(fit-model-simple fit-model-simple.cpp)
target_link_libraries(fit-model-simple eos ${OpenCV_LIBS} ${Boost_LIBRARIES})
target_link_libraries(fit-model-simple "$<$<CXX_COMPILER_ID:GNU>:-pthread>$<$<CXX_COMPILER_ID:Clang>:-pthreads>")
install(TARGETS fit-model-simple DESTINATION bin)

# Model fitting example that fits orthographic camera, shape, blendshapes, and contours:
add_executable(fit-model fit-model.cpp)
target_link_libraries(fit-model eos ${OpenCV_LIBS} ${Boost_LIBRARIES})
target_link_libraries(fit-model "$<$<CXX_COMPILER_ID:GNU>:-pthread>$<$<CXX_COMPILER_ID:Clang>:-pthreads>")
install(TARGETS fit-model DESTINATION bin)

# Generate random samples from the model:
add_executable(generate-obj generate-obj.cpp)
target_link_libraries(generate-obj eos ${OpenCV_LIBS} ${Boost_LIBRARIES} Eigen3::Eigen)
install(TARGETS generate-obj DESTINATION bin)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/data DESTINATION bin)

if(EOS_BUILD_CERES_EXAMPLE)
  # Find Ceres, for the fit-model-ceres app:
  hunter_add_package(ceres-solver)
  find_package(Ceres CONFIG REQUIRED)
  message(STATUS "Ceres locations: Headers: ${CERES_INCLUDE_DIRS} Library: ${CERES_LIBRARIES}")

  # Single and multi-image non-linear model fitting with Ceres example:
  add_executable(fit-model-ceres fit-model-ceres.cpp)
  target_link_libraries(fit-model-ceres eos ${CERES_LIBRARIES} ${OpenCV_LIBS} ${Boost_LIBRARIES})
  install(TARGETS fit-model-ceres DESTINATION bin)
endif()
