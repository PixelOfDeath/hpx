# Copyright (c) 2016 Thomas Heller
# Copyright (c) 2016 Hartmut Kaiser
#
# Distributed under the Boost Software License, Version 1.0. (See accompanying
# file LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt)

set(example_programs)

if(HPX_WITH_CUDA)
  set(example_programs ${example_programs}
      cublas_matmul
      cuda_future
      data_copy
      hello_compute
    )

  set(cublas_matmul_PARAMETERS THREADS_PER_LOCALITY 4)
  set(cuda_future_PARAMETERS THREADS_PER_LOCALITY 4)
  set(data_copy_PARAMETERS THREADS_PER_LOCALITY 4)
  set(hello_compute_PARAMETERS THREADS_PER_LOCALITY 4)

# Append example programs that only compiles with Cuda Clang
  if(HPX_WITH_CUDA_CLANG)
    list(APPEND example_programs
         partitioned_vector
        )
    set(partitioned_vector_PARAMETERS THREADS_PER_LOCALITY 4)
  endif()

  include_directories(${CUDA_INCLUDE_DIRS})

  set(cublas_matmul_FLAGS
      DEPENDENCIES ${CUDA_LIBRARIES} ${CUDA_CUBLAS_LIBRARIES})

  set(cublas_matmul_CUDA OFF)
  set(data_copy_CUDA ON)
  set(hello_compute_CUDA ON)
  set(cuda_future_CUDA OFF)
  set(cuda_future_CUDA_SOURCE trivial_demo)
  set(partitioned_vector_CUDA ON)
  set(partitioned_vector_FLAGS COMPONENT_DEPENDENCIES partitioned_vector)
endif()

foreach(example_program ${example_programs})
  if(${${example_program}_CUDA})
    set(sources
      ${example_program}.cu)
  else()
    set(sources
        ${example_program}.cpp)
  endif()

  if(${example_program}_CUDA_SOURCE)
    set(sources
      ${sources} ${${example_program}_CUDA_SOURCE}.cu)
  endif()

  source_group("Source Files" FILES ${sources})

  # add example executable
  add_hpx_executable(${example_program}
                     SOURCES ${sources}
                     ${${example_program}_FLAGS}
                     FOLDER "Examples/Compute/CUDA")

  # add a custom target for this example
  add_hpx_pseudo_target(examples.compute.cuda.${example_program})

  # make pseudo-targets depend on master pseudo-target
  add_hpx_pseudo_dependencies(examples.compute.cuda
                              examples.compute.cuda.${example_program})

  # add dependencies to pseudo-target
  add_hpx_pseudo_dependencies(examples.compute.cuda.${example_program}
                              ${example_program}_exe)

  if(HPX_WITH_TESTS AND HPX_WITH_TESTS_EXAMPLES)
    add_hpx_example_test("cuda" ${example_program} ${${example_program}_PARAMETERS})
    add_hpx_pseudo_target(tests.examples.compute.cuda.${example_program})
    add_hpx_pseudo_dependencies(tests.examples.compute.cuda
                                tests.examples.compute.cuda.${example_program})
    add_hpx_pseudo_dependencies(tests.examples.compute.cuda.${example_program}
                                ${example_program}_exe)
  endif()
endforeach()
