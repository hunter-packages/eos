cmake_minimum_required(VERSION 3.0)

include("cmake/HunterGate.cmake")
HunterGate(
    URL "https://github.com/ruslo/hunter/archive/v0.15.26.tar.gz"
    SHA1 "7e292fcda36d4cf1214bd4bd8d2360d66fa44395"
)

project(eos VERSION 0.6.1)

set_property(GLOBAL PROPERTY USE_FOLDERS ON)

# Check if a supported compiler is used and add c++11/14 flag
include(CheckCXXCompilerFlag)
if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
  if(CMAKE_CXX_COMPILER_VERSION VERSION_LESS 4.8)
    message(FATAL_ERROR "Need at least gcc 4.8 to compile.")
  elseif(CMAKE_CXX_COMPILER_VERSION VERSION_EQUAL 4.8 OR (CMAKE_CXX_COMPILER_VERSION VERSION_GREATER 4.8 AND CMAKE_CXX_COMPILER_VERSION VERSION_LESS 4.9))
    # This compiles with a warning at the moment, but support for gcc 4.8.x will be removed in the very near future.
    check_cxx_compiler_flag(-std=c++11 HAS_CXX11_FLAG)
    if(HAS_CXX11_FLAG)
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
    endif()
    message(WARNING "Support for gcc 4.8 will be removed in the very near future. Please upgrade your compiler.")
  else() # gcc version is >4.8
    check_cxx_compiler_flag(-std=c++14 HAS_CXX14_FLAG)
    if(HAS_CXX14_FLAG)
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
    endif()
  endif()
  # Note: gcc is fine without -pthreads.
elseif(MSVC) # the quotes are needed here, maybe because "MSVC" seems to be a keyword
  if(CMAKE_CXX_COMPILER_VERSION VERSION_LESS 19)
    message(FATAL_ERROR "Visual Studio 2015 or newer is required.")
  endif()
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
  # >=3.5 works, not sure about older versions. Also, libstdc++ from gcc >=4.9 may be needed.
  if(CMAKE_CXX_COMPILER_VERSION VERSION_LESS 3.5)
    message(WARNING "Clang below version 3.5 may or may not work. Please upgrade your compiler.")
  endif()
  check_cxx_compiler_flag(-std=c++14 HAS_CXX14_FLAG)
  if(HAS_CXX14_FLAG)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
  endif()
  set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -pthreads")
  # Eigen::LevenbergMarquardt probably needs -pthreads.
  # Cleaner way would be to add it to fit-model's target_link_libraries, but that requires a CMake >2.8.10.
else() # no GNU, no MSVC, no Clang
  message(WARNING "You are using an unsupported compiler. Compilation has only been tested with MSVC, GCC and Clang.")
  check_cxx_compiler_flag(-std=c++14 HAS_CXX14_FLAG)
  if(HAS_CXX14_FLAG)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14")
  endif()
endif()

# All the options for building the library. Can be changed on the command-line or in initial_cache.cmake.
message(STATUS "Options:")
option(EOS_BUILD_EXAMPLES "Build the example applications." ON)
message(STATUS "EOS_BUILD_EXAMPLES: ${EOS_BUILD_EXAMPLES}")
option(EOS_BUILD_DOCUMENTATION "Build the library documentation." OFF)
message(STATUS "EOS_BUILD_DOCUMENTATION: ${EOS_BUILD_DOCUMENTATION}")

# Build a CPack driven installer package:
include(InstallRequiredSystemLibraries) # This module will include any runtime libraries that are needed by the project for the current platform
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_LIST_DIR}/LICENSE")
set(CPACK_PACKAGE_VERSION_MAJOR "${eos_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${eos_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${eos_VERSION_PATCH}")
include(CPack)

# Find dependencies:
hunter_add_package(OpenCV)
find_package(OpenCV 2.4.3 REQUIRED core)
message(STATUS "OpenCV include dir found at ${OpenCV_INCLUDE_DIRS}")
message(STATUS "OpenCV lib dir found at ${OpenCV_LIB_DIR}")

hunter_add_package(Boost COMPONENTS system)
find_package(Boost CONFIG REQUIRED system)

hunter_add_package(Eigen)
find_package(Eigen REQUIRED) # Eigen::eigen

hunter_add_package(cereal)
find_package(cereal CONFIG REQUIRED) # cereal::cereal

hunter_add_package(glm)
find_package(glm REQUIRED) # glm

# Header files:
set(HEADERS
  include/eos/core/Landmark.hpp
  include/eos/core/LandmarkMapper.hpp
  include/eos/morphablemodel/PcaModel.hpp
  include/eos/morphablemodel/MorphableModel.hpp
  include/eos/morphablemodel/Blendshape.hpp
  include/eos/morphablemodel/coefficients.hpp
  include/eos/morphablemodel/io/cvssp.hpp
  include/eos/morphablemodel/io/mat_cerealisation.hpp
  include/eos/fitting/affine_camera_estimation.hpp
  include/eos/fitting/nonlinear_camera_estimation.hpp
  include/eos/fitting/detail/nonlinear_camera_estimation_detail.hpp
  include/eos/fitting/detail/optional_cerealisation.hpp
  include/eos/fitting/linear_shape_fitting.hpp
  include/eos/fitting/contour_correspondence.hpp
  include/eos/fitting/blendshape_fitting.hpp
  include/eos/fitting/fitting.hpp
  include/eos/render/Mesh.hpp
  include/eos/render/utils.hpp
  include/eos/render/render.hpp
  include/eos/render/render_affine.hpp
  include/eos/render/detail/render_detail.hpp
  include/eos/render/detail/render_affine_detail.hpp
  include/eos/render/texture_extraction.hpp
  include/eos/render/detail/texture_extraction_detail.hpp
)

# Add header includes:
include_directories("include")
include_directories(${OpenCV_INCLUDE_DIRS})

# Custom target for the library, to make the headers show up in IDEs:
add_custom_target(eos SOURCES ${HEADERS})
source_group(core include/eos/core/*)
source_group(morphablemodel include/eos/morphablemodel/*)
source_group(morphablemodel\\io include/eos/morphablemodel/io/*)
source_group(fitting include/eos/fitting/*)
source_group(fitting\\detail include/eos/fitting/detail/*)
source_group(render include/eos/render/*)
source_group(render\\detail include/eos/render/detail/*)

# The install target:
install(DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/include/ DESTINATION include) # our library headers
install(DIRECTORY ${CMAKE_CURRENT_LIST_DIR}/share/ DESTINATION share) # the model and metadata

if(EOS_BUILD_EXAMPLES)
  add_subdirectory(examples)
endif()

if(EOS_BUILD_DOCUMENTATION)
  add_subdirectory(doc)
endif()
