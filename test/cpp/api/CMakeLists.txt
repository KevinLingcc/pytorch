set(TORCH_API_TEST_DIR "${TORCH_ROOT}/test/cpp/api")
set(TORCH_API_TEST_SOURCES
  ${TORCH_ROOT}/test/cpp/common/main.cpp
  ${TORCH_API_TEST_DIR}/autograd.cpp
  ${TORCH_API_TEST_DIR}/any.cpp
  ${TORCH_API_TEST_DIR}/dataloader.cpp
  ${TORCH_API_TEST_DIR}/enum.cpp
  ${TORCH_API_TEST_DIR}/expanding-array.cpp
  ${TORCH_API_TEST_DIR}/fft.cpp
  ${TORCH_API_TEST_DIR}/functional.cpp
  ${TORCH_API_TEST_DIR}/init.cpp
  ${TORCH_API_TEST_DIR}/integration.cpp
  ${TORCH_API_TEST_DIR}/ivalue.cpp
  ${TORCH_API_TEST_DIR}/jit.cpp
  ${TORCH_API_TEST_DIR}/memory.cpp
  ${TORCH_API_TEST_DIR}/meta_tensor.cpp
  ${TORCH_API_TEST_DIR}/misc.cpp
  ${TORCH_API_TEST_DIR}/module.cpp
  ${TORCH_API_TEST_DIR}/moduledict.cpp
  ${TORCH_API_TEST_DIR}/modulelist.cpp
  ${TORCH_API_TEST_DIR}/modules.cpp
  ${TORCH_API_TEST_DIR}/nested.cpp
  ${TORCH_API_TEST_DIR}/parameterdict.cpp
  ${TORCH_API_TEST_DIR}/parameterlist.cpp
  ${TORCH_API_TEST_DIR}/namespace.cpp
  ${TORCH_API_TEST_DIR}/nn_utils.cpp
  ${TORCH_API_TEST_DIR}/optim.cpp
  ${TORCH_API_TEST_DIR}/ordered_dict.cpp
  ${TORCH_API_TEST_DIR}/rnn.cpp
  ${TORCH_API_TEST_DIR}/sequential.cpp
  ${TORCH_API_TEST_DIR}/transformer.cpp
  ${TORCH_API_TEST_DIR}/serialize.cpp
  ${TORCH_API_TEST_DIR}/special.cpp
  ${TORCH_API_TEST_DIR}/static.cpp
  ${TORCH_API_TEST_DIR}/support.cpp
  ${TORCH_API_TEST_DIR}/tensor_cuda.cpp
  ${TORCH_API_TEST_DIR}/tensor_indexing.cpp
  ${TORCH_API_TEST_DIR}/tensor_options_cuda.cpp
  ${TORCH_API_TEST_DIR}/tensor_options.cpp
  ${TORCH_API_TEST_DIR}/tensor.cpp
  ${TORCH_API_TEST_DIR}/torch_include.cpp
  ${TORCH_API_TEST_DIR}/inference_mode.cpp
  ${TORCH_API_TEST_DIR}/grad_mode.cpp
  ${TORCH_API_TEST_DIR}/operations.cpp
  ${TORCH_API_TEST_DIR}/nested_int.cpp
)
if(USE_CUDA OR USE_ROCM)
  list(APPEND TORCH_API_TEST_SOURCES ${TORCH_API_TEST_DIR}/parallel.cpp)
endif()

add_executable(test_api ${TORCH_API_TEST_SOURCES})
target_include_directories(test_api PRIVATE ${ATen_CPU_INCLUDE})
target_link_libraries(test_api PRIVATE torch gtest_main gmock)

if(USE_CUDA)
  target_compile_definitions(test_api PRIVATE "USE_CUDA")
endif()

if(NOT MSVC)
  # Clang has an unfixed bug leading to spurious missing braces
  # warnings, see https://bugs.llvm.org/show_bug.cgi?id=21629
  target_compile_options_if_supported(test_api "-Wno-missing-braces")
  # Considered to be flaky.  See the discussion at
  # https://github.com/pytorch/pytorch/pull/9608
  target_compile_options_if_supported(test_api "-Wno-maybe-uninitialized")
  # gcc gives nonsensical warnings about variadic.h
  target_compile_options_if_supported(test_api "-Wno-unused-but-set-parameter")

  # Add -Wno-error=nonnull for GCC 12+
  if(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" AND CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 12)
    target_compile_options_if_supported(test_api "-Wno-error=nonnull")
  endif()
endif()

if(INSTALL_TEST)
  set_target_properties(test_api PROPERTIES INSTALL_RPATH "${CMAKE_INSTALL_RPATH}:${_rpath_portable_origin}/../lib")
  install(TARGETS test_api DESTINATION bin)
  # Install PDB files for MSVC builds
  if(MSVC AND BUILD_SHARED_LIBS)
    install(FILES $<TARGET_PDB_FILE:test_api> DESTINATION bin OPTIONAL)
  endif()
endif()

add_executable(parallel_benchmark ${TORCH_API_TEST_DIR}/parallel_benchmark.cpp)
target_include_directories(parallel_benchmark PRIVATE ${ATen_CPU_INCLUDE})
target_link_libraries(parallel_benchmark PRIVATE torch)
