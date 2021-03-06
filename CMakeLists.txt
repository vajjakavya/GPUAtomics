
cmake_minimum_required(VERSION 3.8 FATAL_ERROR)

#project(cmake_and_cuda LANGUAGES CXX CUDA)

set(CMAKE_C_COMPILER /usr/bin/gcc-6)
set(CMAKE_CXX_COMPILER /usr/bin/g++-6)
set(CMAKE_CUDA_COMPILER /usr/local/cuda/bin/nvcc)
set(CMAKE_CUDA_FLAGS "-arch=sm_70 -ccbin=/usr/bin/g++-6")

project(cmake_and_cuda LANGUAGES CXX CUDA)



include(CTest)

add_library(sandboxlib STATIC
  kernel.cu
  kernel.h
 )

# Request that particles be built with -std=c++11
# As this is a public compile feature anything that links to particles
# will also build with -std=c++11
target_compile_features(sandboxlib PUBLIC cxx_std_11)

# We need to explicitly state that we need all CUDA files in the particle
# library to be built with -dc as the member functions could be called by
# other libraries and executables
set_target_properties( sandboxlib
                       PROPERTIES CUDA_SEPARABLE_COMPILATION ON
                      )

if(BUILD_TESTING)

  add_executable(sandbox sandbox.cu)

  set_target_properties(sandbox PROPERTIES CUDA_SEPARABLE_COMPILATION ON)
  target_link_libraries(sandbox PRIVATE sandboxlib)

  #add_test(NAME particles_10k COMMAND particle_test 10000 )
  #add_test(NAME particles_256k COMMAND particle_test 256000 )

  if(APPLE)
    # We need to add the default path to the driver (libcuda.dylib) as an rpath,
    # so that the static cuda runtime can find it at runtime.
    set_property(TARGET particle_test PROPERTY BUILD_RPATH ${CMAKE_CUDA_IMPLICIT_LINK_DIRECTORIES})
  endif()
endif()



