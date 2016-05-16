cmake_minimum_required(VERSION 2.8)

project(tiny-process-library)

set(CMAKE_DEBUG_POSTFIX _d)
if(MSVC)
  add_definitions(-D_CRT_SECURE_NO_WARNINGS)
else()
  if(CMAKE_VERSION VERSION_LESS 3.1)
    set(gnu_cxx11_flag -std=c++11)
  elseif(NOT CMAKE_CXX_STANDARD)
    set(CMAKE_CXX_STANDARD 11)
  endif()
  if(CMAKE_CXX_COMPILER_ID MATCHES GNU OR CMAKE_CXX_COMPILER_ID MATCHES Clang)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${gnu_cxx11_flag} -Wall -Wextra -Wno-unused-parameter")
  endif()
endif()

set(CMAKE_THREAD_PREFER_PTHREAD 1)
set(THREADS_PREFER_PTHREAD_FLAG 1)
find_package(Threads REQUIRED)

set(process_source_files process.cpp)

if(WIN32)
  list(APPEND process_source_files process_win.cpp)
  #If compiled using MSYS2, use sh to run commands
  if(MSYS)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DMSYS_PROCESS_USE_SH")
  endif()
else()
  list(APPEND process_source_files process_unix.cpp)
endif()

add_library(tiny-process-library ${process_source_files})
add_executable(examples examples.cpp)

if(CMAKE_VERSION VERSION_LESS 3.1)
  set(threads ${THREAD_LIBS_INIT})
else()
  set(threads Threads::Threads)
endif()

target_link_libraries(tiny-process-library ${threads})
target_link_libraries(examples tiny-process-library)

# To enable tests: cmake -DENABLE_TESTING=1 .
if(ENABLE_TESTING)
  if(NOT CMAKE_CROSSCOMPILING)
    enable_testing()
  endif()
  add_subdirectory(test)
endif()

if(NOT CMAKE_VERSION VERSION_LESS 2.8.11)
  install(TARGETS tiny-process-library EXPORT tiny-process-library-targets
      ARCHIVE DESTINATION lib
      LIBRARY DESTINATION lib
      RUNTIME DESTINATION bin)
  install(EXPORT tiny-process-library-targets
    DESTINATION lib/cmake/tiny-process-library
    NAMESPACE ::)
  configure_file(tiny-process-library-config.cmake.in
    hide/tiny-process-library-config.cmake @ONLY)
  install(
    FILES ${CMAKE_CURRENT_BINARY_DIR}/hide/tiny-process-library-config.cmake
    DESTINATION lib/cmake/tiny-process-library)
  install(FILES process.hpp
    DESTINATION include/tiny-process-library)
  target_include_directories(tiny-process-library PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:include/tiny-process-library>)
endif()
