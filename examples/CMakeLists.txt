project(examples)
cmake_minimum_required(VERSION 2.8.10)

# The examples need a few additional dependencies (e.g. boost filesystem and OpenCV highgui):

#check installed version in order to include the correct OpenCV libraries
#version variable is defined from project root's CMakeLists
if("${OpenCV_VERSION_MAJOR}$" EQUAL 2)
  message(STATUS "OpenCV 2.x detected")
  find_package(OpenCV 2.4.3 REQUIRED core imgproc highgui)
elseif("${OpenCV_VERSION_MAJOR}$" EQUAL 3)
  message(STATUS "OpenCV 3.x detected - including imgcodecs for compatibility")
  find_package(OpenCV 3.0.0 REQUIRED core imgproc imgcodecs)
endif()
message(STATUS "OpenCV include dir found at ${OpenCV_INCLUDE_DIRS}")
message(STATUS "OpenCV lib dir found at ${OpenCV_LIB_DIR}")
# This allows us to compile in RelWithDebInfo. It'll use the Release-version of OpenCV:
set_target_properties(${OpenCV_LIBS} PROPERTIES MAP_IMPORTED_CONFIG_RELWITHDEBINFO RELEASE)

hunter_add_package(Boost COMPONENTS system filesystem program_options)
find_package(Boost CONFIG REQUIRED system filesystem program_options)

if(MSVC)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /bigobj")
endif()

# Model fitting (affine cam & shape to landmarks) example:
add_executable(fit-model fit-model.cpp)
target_link_libraries(fit-model ${OpenCV_LIBS} Boost::system Boost::filesystem Boost::program_options)

# Converts a CVSSP .scm Morphable Model to a cereal binary file:
add_executable(scm-to-cereal scm-to-cereal.cpp)
target_link_libraries(scm-to-cereal ${OpenCV_LIBS} Boost::system Boost::filesystem Boost::program_options)

# install target:
install(TARGETS fit-model DESTINATION bin)
install(TARGETS scm-to-cereal DESTINATION bin)
install(DIRECTORY ${CMAKE_SOURCE_DIR}/examples/data DESTINATION bin)
