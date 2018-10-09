project(CASCEXTRACTOR)
cmake_minimum_required(VERSION 2.6)

set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CASCEXTRACTOR_BINARY_DIR}/bin")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CASCEXTRACTOR_BINARY_DIR}/lib")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CASCEXTRACTOR_BINARY_DIR}/bin")


if (NOT EXISTS "${CASCEXTRACTOR_SOURCE_DIR}/CascLib/CMakeLists.txt")
    message(FATAL_ERROR
"Missing dependency: CascLib
CASCExtractor requires the CascLib library.
It is provided as a GIT submodule of this repository.
Did you forgot to execute the following commands?
   git submodule init
   git submodule update")
endif()


if (WIN32)
	set(CASC_BUILD_SHARED_LIB OFF CACHE BOOL "Build CASC in static mode to link on windows")
	set(CASC_BUILD_STATIC_LIB ON CACHE BOOL "Build CASC in static mode to link on windows")
endif()

add_subdirectory(CascLib)

if (WIN32)
	set_target_properties(casc_static PROPERTIES OUTPUT_NAME CascLibRAD)
endif()

include_directories("${CASCEXTRACTOR_SOURCE_DIR}/CascLib/src/"
                    "${CASCEXTRACTOR_SOURCE_DIR}/include/"
)

# Replacement for dirent.h and unistd.h
if (WIN32)
  include_directories("${CMAKE_CURRENT_SOURCE_DIR}/dirent/include")
endif()

add_executable(CASCExtractor main.cpp)
target_link_libraries(CASCExtractor casc_static)

# Set the RPATH
if (APPLE)
    set_target_properties(CASCExtractor PROPERTIES LINK_FLAGS "-Wl,-rpath,@loader_path/.")
elseif (UNIX)
    set_target_properties(CASCExtractor PROPERTIES INSTALL_RPATH ".")
endif()
