# The python bindings may need a few additional dependencies (e.g. boost filesystem and OpenCV), which are
# included below - at some point it should be checked if they are really needed.

hunter_add_package(OpenCV)
find_package(OpenCV REQUIRED core imgproc imgcodecs)

hunter_add_package(Boost COMPONENTS system filesystem program_options)
find_package(Boost CONFIG REQUIRED system filesystem program_options)
set(boost_libs Boost::system Boost::filesystem Boost::program_options)

pybind11_add_module(python-bindings generate-python-bindings.cpp pybind11_glm.hpp pybind11_opencv.hpp)
target_link_libraries(python-bindings PRIVATE eos ${OpenCV_LIBS} ${boost_libs})
set_target_properties(python-bindings PROPERTIES OUTPUT_NAME eos)

install(TARGETS python-bindings DESTINATION python)
install(FILES ${CMAKE_CURRENT_SOURCE_DIR}/demo.py DESTINATION python)
