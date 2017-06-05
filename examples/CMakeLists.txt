# The examples need a few additional dependencies (e.g. boost filesystem and OpenCV highgui):

hunter_add_package(OpenCV)
find_package(OpenCV REQUIRED core imgproc imgcodecs)

hunter_add_package(Boost COMPONENTS system filesystem program_options)
find_package(Boost CONFIG REQUIRED system filesystem program_options)
set(boost_libs Boost::system Boost::filesystem Boost::program_options)

# https://stackoverflow.com/a/39547577
set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
set(THREADS_PREFER_PTHREAD_FLAG TRUE)
find_package(Threads REQUIRED)

# Simple model fitting (orthographic camera & shape to landmarks) example:
add_executable(fit-model-simple fit-model-simple.cpp)
target_link_libraries(fit-model-simple eos ${OpenCV_LIBS} ${boost_libs})
target_link_libraries(fit-model-simple Threads::Threads)
install(TARGETS fit-model-simple DESTINATION bin)

# Model fitting example that fits orthographic camera, shape, blendshapes, and contours:
add_executable(fit-model fit-model.cpp)
target_link_libraries(fit-model eos ${OpenCV_LIBS} ${boost_libs})
target_link_libraries(fit-model Threads::Threads)
install(TARGETS fit-model DESTINATION bin)

# Generate random samples from the model:
add_executable(generate-obj generate-obj.cpp)
target_link_libraries(generate-obj eos ${OpenCV_LIBS} ${boost_libs} Eigen3::Eigen)
install(TARGETS generate-obj DESTINATION bin)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/data DESTINATION bin)

if(EOS_BUILD_CERES_EXAMPLE)
  # Find Ceres, for the fit-model-ceres app:
  hunter_add_package(ceres-solver)
  find_package(Ceres CONFIG REQUIRED)
  message(STATUS "Ceres locations: Headers: ${CERES_INCLUDE_DIRS} Library: ${CERES_LIBRARIES}")

  # Single and multi-image non-linear model fitting with Ceres example:
  add_executable(fit-model-ceres fit-model-ceres.cpp)
  target_link_libraries(fit-model-ceres eos ceres ${OpenCV_LIBS} ${boost_libs})
  install(TARGETS fit-model-ceres DESTINATION bin)
endif()
