# Sets up a build target for the Voxelizer that makes a library with CUDA.
# cuda_add_library( Voxelizer STATIC ${SOURCE_FILES} ${HEADER_FILES} )

add_executable( VTester ${TEST_SOURCES} ${TEST_HEADERS} )

add_dependencies( VTester Voxelizer )

if ( WIN32 )
target_link_libraries( VTester optimized;Voxelizer.lib;debug;Voxelizer_d.lib;${CUDA_LIBRARIES};${VTK_LIBRARIES};${Boost_LIBRARIES} )
elseif ( UNIX )
target_link_libraries( VTester optimized;libVoxelizer.a;debug;libVoxelizer_d.a;${CUDA_LIBRARIES};${VTK_LIBRARIES};${Boost_LIBRARIES} )
endif ()

# Copies the library to <BuildDir>/lib for easier access without an install.
set_target_properties( VTester PROPERTIES RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin/ )