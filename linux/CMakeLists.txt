# The Flutter tooling requires that developers have CMake 3.10 or later
# installed. You should not increase this version, as doing so will cause
# the plugin to fail to compile for some customers of the plugin.
cmake_minimum_required(VERSION 3.10)
cmake_policy(SET CMP0168 OLD)
cmake_policy(SET CMP0169 OLD)

# Project-level configuration.
set(PROJECT_NAME "lcpp_ngin")
project(${PROJECT_NAME} LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(NOT DEFINED CMAKE_INSTALL_PREFIX)
  set(CMAKE_INSTALL_PREFIX "${CMAKE_CURRENT_BINARY_DIR}" CACHE PATH "Needed to avoid cmake_install.cmake build issues." FORCE)
endif()

find_package(Vulkan REQUIRED COMPONENTS glslc)
find_package(OpenMP REQUIRED)
find_package(cpuinfo REQUIRED)

# set(CMAKE_INSTALL_PREFIX "${CMAKE_BINARY_DIR}/dist" CACHE STRING "Needed by cpuinfo and duckdb" FORCE)
set(CMAKE_BUILD_RPATH_USE_ORIGIN TRUE)
set(LLAMACPP_SRC_DIR ${CMAKE_CURRENT_SOURCE_DIR}/../src)

set(BUILD_SHARED_LIBS ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17 -Wall -Wno-error -fPIC -msse2 -msse -mavx -mavx2 -mfma")

# Set the linker flags for shared libraries
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl,--build-id=none")

################## llama.cpp
FetchContent_Declare(
  llama_cpp
  GIT_REPOSITORY https://github.com/ggml-org/llama.cpp.git
  GIT_TAG        4a4f426944e79b79e389f9ed7b34831cb9b637ad # release-1.10.0
  GIT_SHALLOW    TRUE
  
  EXCLUDE_FROM_ALL
)

##########################  kleidiai
FetchContent_Declare(
  kleidiai
  GIT_REPOSITORY https://git.gitlab.arm.com/kleidi/kleidiai.git
  GIT_TAG        v1.11.0 # release-1.10.0
  GIT_SHALLOW    TRUE
  
  EXCLUDE_FROM_ALL
  
  CMAKE_ARGS -DCMAKE_POSITION_INDEPENDENT_CODE=ON
             -DKLEIDIAI_BUILD_TESTS=OFF
             -DKLEIDIAI_BUILD_BENCHMARK=OFF
			 -DKLEIDIAI_ENABLE_CLANG_TIDY=OFF
			 -DBUILD_SHARED_LIBS=ON
)


set(KLEIDIAI_BUILD_TESTS       OFF CACHE BOOL "Build unit tests."             FORCE)
set(KLEIDIAI_BUILD_BENCHMARK   OFF CACHE BOOL "Build the benchmark tool."     FORCE)
set(KLEIDIAI_ENABLE_CLANG_TIDY OFF CACHE BOOL "Build with Clang-Tidy checks." FORCE)

FetchContent_MakeAvailable(kleidiai)

######################## llama.cpp cont

# Set the linker flags for shared libraries
set(LLAMA_BUILD_COMMON ON CACHE BOOL "llama: build common utils library" FORCE)
set(LLAMA_CURL OFF CACHE BOOL "llama: use libcurl to download model from an URL" FORCE)
set(LLAMA_LLGUIDANCE ON CACHE BOOL "llama-common: include LLGuidance library for structured output in common utils" FORCE)
set(LLAMA_OPENSSL OFF CACHE BOOL "llama: use openssl to support HTTPS" FORCE)
set(LLAMA_HTTPLIB OFF CACHE BOOL "llama: if libcurl is disabled, use httplib to download model from an URL" FORCE)
set(LLAMA_TOOLS_INSTALL OFF CACHE BOOL "llama: install tools" FORCE)
set(LLAMA_BUILD_TESTS    OFF CACHE BOOL "llama: build tests"          FORCE)
set(LLAMA_BUILD_TOOLS    OFF CACHE BOOL "llama: build tools"          FORCE)
set(LLAMA_BUILD_SERVER   OFF CACHE BOOL "llama: build server example" FORCE)
set(LLAMA_BUILD_EXAMPLES OFF CACHE BOOL "llama: build examples"       FORCE)

################## ggml settings ###############################
set(BLA_VENDOR "OpenBLAS" CACHE STRING "BLAS Vendor" FORCE)
set(GGML_BACKEND_DL ON CACHE BOOL "ggml: build backends as dynamic libraries (requires BUILD_SHARED_LIBS)" FORCE)
set(GGML_BLAS_VENDOR "OpenBLAS" CACHE STRING "BLAS Vendor" FORCE)
set(GGML_CPU     ON CACHE BOOL "ggml: enable CPU backend" FORCE)
set(GGML_CPU_REPACK ON CACHE BOOL "ggml: use runtime weight conversion of Q4_0 to Q4_X_X" FORCE)
set(GGML_NATIVE OFF CACHE BOOL "llama: disable -march=native flag" FORCE)
set(GGML_CPU_ALL_VARIANTS OFF CACHE BOOL "ggml: build all variants of the CPU backend (requires GGML_BACKEND_DL)" FORCE)
set(GGML_VULKAN ON CACHE BOOL "llama: enable vulkan" FORCE)
set(GGML_AVX    ON CACHE BOOL "ggml: enable AVX" FORCE)
set(GGML_AVX2   ON CACHE BOOL "ggml: enable AVX2" FORCE)
set(GGML_FMA   ON CACHE BOOL "ggml: enable FMA" FORCE)
set(GGML_F16C  ON CACHE BOOL "ggml: enable F16C" FORCE)
set(GGML_BLAS ON CACHE BOOL "ggml: use BLAS" FORCE)
set(GGML_OPENCL ON CACHE BOOL "ggml: use OpenCL" FORCE)
set(GGML_OPENCL_EMBED_KERNELS ON CACHE BOOL "ggml: embed kernels" FORCE)
set(GGML_LLAMAFILE OFF CACHE BOOL "ggml: use LLAMAFILE" FORCE)
set(GGML_KOMPUTE OFF CACHE BOOL "ggml: use Kompute" FORCE)
set(GGML_CUDA ON CACHE BOOL "ggml: use CUDA" FORCE)
set(GGML_CUDA_FA ON CACHE BOOL "ggml: compile ggml FlashAttention CUDA kernels" FORCE)
set(GGML_HIP OFF CACHE BOOL "ggml: use HIP" FORCE)
set(GGML_HIP_MMQ_MFMA ON CACHE BOOL "ggml: enable MFMA MMA for CDNA in MMQ" FORCE)
set(GGML_OPENMP ON CACHE BOOL "ggml: use OpenMP" FORCE)
set(GGML_CPU_KLEIDIAI  ON CACHE BOOL "ggml: use KleidiAI optimized kernels if applicable" FORCE)


FetchContent_MakeAvailable(llama_cpp)

add_subdirectory("${LLAMACPP_SRC_DIR}" "${CMAKE_BINARY_DIR}/shared")

set(lcpp_ngin_bundled_libraries
	$<TARGET_FILE:lcpp_ngin>
	$<TARGET_FILE:llama>
	$<TARGET_FILE:llguidance>
	$<TARGET_FILE:ggml-cuda>
	# $<TARGET_FILE:ggml-hip>
	$<TARGET_FILE:ggml-vulkan>
	$<TARGET_FILE:ggml-opencl>
	$<TARGET_FILE:ggml-blas>
	$<TARGET_FILE:ggml-cpu>
	$<TARGET_FILE:ggml-base>
	$<TARGET_FILE:ggml>
    PARENT_SCOPE
)
