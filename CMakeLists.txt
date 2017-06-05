cmake_minimum_required(VERSION 3.1.3)

include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.19.0.tar.gz"
    SHA1 "e9d11aee66607cfcfbec5e3811d4804135b5bfac"
)  

project(eos VERSION 0.12.1)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# This sets the C++ standard to c++14 and required for all the following targets that we define.
# It has no effect on MSVC though - we thus define more specific requirements for each executable target respectively.
# Also it will not apply to the eos library target, since it is an INTERFACE_LIBRARY, and these properties do not apply to interface libraries.
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF) # This makes CMake use -std=c++11 instead of -std=gnu++11
# This list is likely not complete, but it should be sufficient to error out on old compilers that we cannot build on:
set(EOS_CXX_COMPILE_FEATURES
  cxx_defaulted_functions
  cxx_generalized_initializers
  cxx_generic_lambdas
  cxx_lambdas
  cxx_nonstatic_member_init
  cxx_range_for
  cxx_right_angle_brackets
  cxx_strong_enums)

# All the options for building the library. Can be changed on the command-line or in initial_cache.cmake.
message(STATUS "Options:")
option(EOS_BUILD_EXAMPLES "Build the example applications." ON)
message(STATUS "EOS_BUILD_EXAMPLES: ${EOS_BUILD_EXAMPLES}")
option(EOS_BUILD_CERES_EXAMPLE "Build the fit-model-ceres example (requires Ceres)." OFF)
message(STATUS "EOS_BUILD_CERES_EXAMPLE: ${EOS_BUILD_CERES_EXAMPLE}")
option(EOS_BUILD_UTILS "Build utility applications." OFF)
message(STATUS "EOS_BUILD_UTILS: ${EOS_BUILD_UTILS}")
option(EOS_BUILD_DOCUMENTATION "Build the library documentation." OFF)
message(STATUS "EOS_BUILD_DOCUMENTATION: ${EOS_BUILD_DOCUMENTATION}")
option(EOS_GENERATE_PYTHON_BINDINGS "Build python bindings. Requires python to be installed." OFF)
message(STATUS "EOS_GENERATE_PYTHON_BINDINGS: ${EOS_GENERATE_PYTHON_BINDINGS}")
option(EOS_GENERATE_MATLAB_BINDINGS "Build Matlab bindings. Requires Matlab with the compiler installed or the Matlab Compiler Runtime." OFF)
message(STATUS "EOS_GENERATE_MATLAB_BINDINGS: ${EOS_GENERATE_MATLAB_BINDINGS}")

# Build a CPack driven installer package:
include(InstallRequiredSystemLibraries) # This module will include any runtime libraries that are needed by the project for the current platform
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
set(CPACK_PACKAGE_VERSION_MAJOR "${eos_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${eos_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${eos_VERSION_PATCH}")
include(CPack)

hunter_add_package(OpenCV)
find_package(OpenCV REQUIRED core)

hunter_add_package(Boost COMPONENTS system)
find_package(Boost CONFIG REQUIRED system)
message(STATUS "Boost found at ${Boost_INCLUDE_DIRS}")

hunter_add_package(Eigen)
find_package(Eigen3 CONFIG REQUIRED)

hunter_add_package(cereal)
find_package(cereal CONFIG REQUIRED)

hunter_add_package(glm)
find_package(glm REQUIRED)

hunter_add_package(nanoflann)
find_package(nanoflann CONFIG REQUIRED)

hunter_add_package(eigen3-nnls)
find_package(eigen3-nnls CONFIG REQUIRED)

# Header files of the eos library:
set(eos_headers
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/core/Landmark.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/core/LandmarkMapper.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/core/Mesh.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/morphablemodel/PcaModel.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/morphablemodel/MorphableModel.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/morphablemodel/Blendshape.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/morphablemodel/coefficients.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/morphablemodel/EdgeTopology.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/morphablemodel/io/cvssp.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/morphablemodel/io/mat_cerealisation.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/fitting/affine_camera_estimation.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/fitting/orthographic_camera_estimation_linear.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/fitting/nonlinear_camera_estimation.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/fitting/detail/nonlinear_camera_estimation_detail.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/fitting/detail/optional_cerealisation.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/fitting/detail/glm_cerealisation.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/fitting/linear_shape_fitting.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/fitting/contour_correspondence.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/fitting/blendshape_fitting.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/fitting/closest_edge_fitting.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/fitting/fitting.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/fitting/ceres_nonlinear.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/fitting/RenderingParameters.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/render/utils.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/render/render.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/render/render_affine.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/render/detail/render_detail.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/render/detail/render_affine_detail.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/render/texture_extraction.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/render/detail/texture_extraction_detail.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/render/SoftwareRenderer.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/render/VertexShader.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/render/Rasterizer.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/render/FragmentShader.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/render/detail/Vertex.hpp
  ${CMAKE_CURRENT_SOURCE_DIR}/include/eos/video/Keyframe.hpp
)

add_library(eos INTERFACE)
target_link_libraries(
    eos
    INTERFACE
    ${OpenCV_LIBS}
    Boost::system
    Eigen3::Eigen
    cereal::cereal
    glm
    eigen3-nnls::eigen3-nnls
    nanoflann::nanoflann
    )

target_sources(eos INTERFACE ${eos_headers})  

target_compile_features(eos INTERFACE ${EOS_CXX_COMPILE_FEATURES})

# Add header includes:
set(eos_inc_dir "${CMAKE_CURRENT_LIST_DIR}/include")
target_include_directories(eos
  INTERFACE
  "$<BUILD_INTERFACE:${eos_inc_dir}>" # i.e., submodule use
  )

# Custom target for the library, to make the headers show up in IDEs:
add_custom_target(eos-headers SOURCES ${eos_headers})
source_group(core REGULAR_EXPRESSION include/eos/core/*)
source_group(morphablemodel REGULAR_EXPRESSION include/eos/morphablemodel/*)
source_group(morphablemodel\\io REGULAR_EXPRESSION include/eos/morphablemodel/io/*)
source_group(fitting REGULAR_EXPRESSION include/eos/fitting/*)
source_group(fitting\\detail REGULAR_EXPRESSION include/eos/fitting/detail/*)
source_group(render REGULAR_EXPRESSION include/eos/render/*)
source_group(render\\detail REGULAR_EXPRESSION include/eos/render/detail/*)
source_group(video REGULAR_EXPRESSION include/eos/video/*)

# The eos install target:
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/include/ DESTINATION include) # our library headers
install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/share/ DESTINATION share) # the model and metadata

if(EOS_BUILD_EXAMPLES)
  add_subdirectory(examples)
endif()

if(EOS_BUILD_UTILS)
  add_subdirectory(utils)
endif()

if(EOS_BUILD_DOCUMENTATION)
  add_subdirectory(doc)
endif()

if(EOS_GENERATE_PYTHON_BINDINGS)
  set(PYBIND11_PATH "${eos_3RDPARTY_DIR}/pybind11")
  # If this fails, the repo has probably not been cloned with submodules. Run: git submodule update --init
  add_subdirectory(${PYBIND11_PATH}) # add and initialise pybind11
  add_subdirectory(python) # the actual bindings
  install(DIRECTORY ${eos_3RDPARTY_DIR}/pybind11/include/ DESTINATION 3rdparty/pybind11/include) # pybind11 headers
  install(FILES ${eos_3RDPARTY_DIR}/pybind11/LICENSE DESTINATION 3rdparty/pybind11/) # pybind11 licence
endif()

if(EOS_GENERATE_MATLAB_BINDINGS)
  add_subdirectory(matlab)
  install(DIRECTORY ${eos_3RDPARTY_DIR}/mexplus/include/ DESTINATION 3rdparty/mexplus/include) # mexplus headers
  install(FILES ${eos_3RDPARTY_DIR}/mexplus/LICENSE DESTINATION 3rdparty/mexplus/) # mexplus licence
endif()
