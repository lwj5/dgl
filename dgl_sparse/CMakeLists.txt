cmake_minimum_required(VERSION 3.5)
project(dgl_sparse C CXX)

# Find PyTorch cmake files and PyTorch versions with the python interpreter $PYTHON_INTERP
# ("python3" or "python" if empty)
if(NOT PYTHON_INTERP)
  find_program(PYTHON_INTERP NAMES python3 python)
endif()
message(STATUS "Using Python interpreter: ${PYTHON_INTERP}")
file(TO_NATIVE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/find_cmake.py FIND_CMAKE_PY)
execute_process(
  COMMAND ${PYTHON_INTERP} ${FIND_CMAKE_PY}
  OUTPUT_VARIABLE TORCH_PREFIX_VER
  OUTPUT_STRIP_TRAILING_WHITESPACE)
message(STATUS "find_cmake.py output: ${TORCH_PREFIX_VER}")
list(GET TORCH_PREFIX_VER 0 TORCH_PREFIX)
list(GET TORCH_PREFIX_VER 1 TORCH_VER)
message(STATUS "Configuring for PyTorch ${TORCH_VER}")

if(USE_CUDA)
  add_definitions(-DDGL_USE_CUDA)
endif()

set(Torch_DIR "${TORCH_PREFIX}/Torch")
message(STATUS "Setting directory to ${Torch_DIR}")
find_package(Torch REQUIRED)

set(LIB_DGL_SPARSE_NAME "dgl_sparse_pytorch_${TORCH_VER}")

set(SPARSE_DIR "${CMAKE_SOURCE_DIR}/src")
set(SPARSE_INCLUDE "${CMAKE_SOURCE_DIR}/include")
file(GLOB SPARSE_HEADERS ${SPARSE_INCLUDE})
file(GLOB SPARSE_SRC
  ${SPARSE_DIR}/*.cc
)
add_library(${LIB_DGL_SPARSE_NAME} SHARED ${SPARSE_SRC} ${SPARSE_HEADERS})
target_include_directories(
  ${LIB_DGL_SPARSE_NAME} PRIVATE ${SPARSE_DIR} ${SPARSE_HEADERS})
target_link_libraries(${LIB_DGL_SPARSE_NAME} "${TORCH_LIBRARIES}")

add_subdirectory("${CMAKE_SOURCE_DIR}/../third_party/dmlc-core" "${CMAKE_SOURCE_DIR}/build/third_party/dmlc-core")
target_include_directories(${LIB_DGL_SPARSE_NAME} PRIVATE "${CMAKE_SOURCE_DIR}/../third_party/dmlc-core/include")
target_link_libraries(${LIB_DGL_SPARSE_NAME} dmlc)
set(GOOGLE_TEST 0) # Turn off dmlc-core test
