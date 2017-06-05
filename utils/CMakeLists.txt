# The utils need a few additional dependencies (e.g. boost filesystem and OpenCV highgui):

hunter_add_package(OpenCV)
find_package(OpenCV REQUIRED core imgproc imgcodecs)

hunter_add_package(Boost COMPONENTS system filesystem program_options)
find_package(Boost CONFIG REQUIRED system filesystem program_options)
set(boost_libs Boost::system Boost::filesystem Boost::program_options)

# Converts a CVSSP .scm Morphable Model to a cereal binary file:
add_executable(scm-to-cereal scm-to-cereal.cpp)
target_link_libraries(scm-to-cereal eos ${OpenCV_LIBS} ${boost_libs})

# Converts a file created with share/convert_bfm2009_to_raw_binary.m to a cereal binary file:
add_executable(bfm-binary-to-cereal bfm-binary-to-cereal.cpp)
target_link_libraries(bfm-binary-to-cereal eos ${OpenCV_LIBS} ${boost_libs})

# Reads an edgestruct CSV file created from Matlab, and converts it to json:
add_executable(edgestruct-csv-to-json edgestruct-csv-to-json.cpp)
target_link_libraries(edgestruct-csv-to-json eos ${boost_libs})

# install target:
install(TARGETS scm-to-cereal DESTINATION bin)
install(TARGETS bfm-binary-to-cereal DESTINATION bin)
install(TARGETS edgestruct-csv-to-json DESTINATION bin)
