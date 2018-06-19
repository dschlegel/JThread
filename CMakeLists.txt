cmake_minimum_required(VERSION 3.1)

project(JThread VERSION 1.3.3 LANGUAGES CXX)

include(CheckLibraryExists)
include(GNUInstallDirs)

find_package(Threads)
if(NOT CMAKE_USE_WIN32_THREADS_INIT AND NOT CMAKE_USE_PTHREADS_INIT)
    message(FATAL_ERROR "Can find neither pthread support nor Win32 thread support")
endif()

add_subdirectory(src)
