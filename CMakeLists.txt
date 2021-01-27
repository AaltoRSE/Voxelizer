cmake_minimum_required( VERSION 2.8 )

project( Voxelizer )

# Prints more information about the compilation process.
set( CMAKE_VERBOSE_MAKEFILE ON )

# Set the C++ standard to 11 and disable Boost auto-link for the system libarary
# Both seem to be needed to link against the Anaconda Boost library
set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DBOOST_SYSTEM_NO_LIB -std=gnu++11")


# Collect all relevant files to be sent to the compilers.
# Also includes these files into the project for easy access.
# If new files are added, the project files need to be regenerated with cmake.
file( GLOB SOURCE_FILES . src/*.cpp src/*.cu )
file( GLOB HEADER_FILES . include/*.h )

# Limits the configuration types available. Doesn't seem to work, though...
set( CMAKE_CONFIGURATION_TYPES Release Debug )

if ( UNIX )
	#option( CMAKE_BUILD_SHARED_LIBS	"Choose to build either shared or static libraries." OFF )
	set( CMAKE_BUILD_TYPE Release CACHE STRING "Build type (Debug/Release) for single-configuration build systems." )
endif ()

# Boost options.
set( Boost_USE_STATIC_LIBS ON )
set( Boost_USE_MULTITHREADED ON )
set( Boost_USE_STATIC_RUNTIME OFF )

# Require Boost >= 1.53 with system and thread components.
find_package( Boost 1.53 REQUIRED COMPONENTS system thread  date_time )

# If boost cannot be found, make a cache variable that allows the user to
# specify the Boost root directory where the libraries and includes are found.
if ( Boost_FOUND )
	include_directories( ${Boost_INCLUDE_DIRS} )
else ()
	set( BOOST_ROOT "" CACHE PATH "Path to Boost root directory." )
endif ()

# Require CUDA >= 5.0
find_package( CUDA 5.0 REQUIRED )

# CUDA 5.0 doesn't support gcc 4.7 and above.
if ( UNIX AND (CUDA_VERSION VERSION_LESS "5.5") )
	if ( CMAKE_CXX_COMPILER_ID STREQUAL "GNU" )
		if ( NOT (CMAKE_COMPILER_VERSION VERSION_LESS "4.7") )
			message( FATAL_ERROR "For CUDA 5.5, gcc/g++ needs to be < 4.7" )
		endif()
	endif()
endif ()

# CUDA 5.5 doesn't support gcc 4.8 and above.
if( UNIX AND (CUDA_VERSION VERSION_EQUAL "5.5") )
	if ( CMAKE_CXX_COMPILER_ID STREQUAL "GNU" )
		if ( NOT (CMAKE_COMPILER_VERSION VERSION_LESS "4.8") )
			message( FATAL_ERROR "For CUDA 5.5, gcc/g++ needs to be < 4.8" )
		endif()
	endif()
endif()

option( GENERATE_DOXYGEN_DOCS "Enables or disables the generation of documentation with Doxygen." OFF )
option( GENERATE_TESTS "Adds the compilation of a test program as a target." OFF )

# Configuration-dependent settings.
set( CMAKE_DEBUG_POSTFIX "_d" )



# Set CUDA architecture
if(CUDA_GENCODE)
  set( CUDA_NVCC_FLAGS_RELEASE ${CUDA_NVCC_FLAGS_RELEASE};
                            -gencode ${CUDA_GENCODE}
                         )

  set( CUDA_NVCC_FLAGS_DEBUG ${CUDA_NVCC_FLAGS_DEBUG};
                            -gencode ${CUDA_GENCODE}
                         )
else()
  set( CUDA_NVCC_FLAGS_RELEASE ${CUDA_NVCC_FLAGS_RELEASE};
	                          -gencode arch=compute_60,code=sm_70
	                          -gencode arch=compute_61,code=sm_61
	                          -gencode arch=compute_60,code=sm_60
	                          -gencode arch=compute_52,code=sm_52
	                          -gencode arch=compute_37,code=sm_37
                         )
  set( CUDA_NVCC_FLAGS_DEBUG ${CUDA_NVCC_FLAGS_DEBUG};
	                          -gencode arch=compute_60,code=sm_70
	                          -gencode arch=compute_61,code=sm_61
	                          -gencode arch=compute_60,code=sm_60
	                          -gencode arch=compute_52,code=sm_52
	                          -gencode arch=compute_37,code=sm_37
                         )
endif()


# Unix-specific compiler flags.
if ( UNIX )
set( CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -w -m64 -std=gnu++11 -fPIC" )
endif ()

#message( STATUS "R Flags: ${CMAKE_CXX_FLAGS_RELEASE}" )

# Include files from the subdirectories and process the CMakeLists.txt in those
# directories.
include_directories( include )
add_subdirectory( src )

if ( GENERATE_DOXYGEN_DOCS )
	add_subdirectory( doc )
endif ()

if ( GENERATE_TESTS )
	add_subdirectory( tests )
endif ()
