cmake_minimum_required(VERSION 3.1)

project(JThread VERSION 1.3.3 LANGUAGES CXX)

include(CheckLibraryExists)
include(GNUInstallDirs)

find_package(Threads QUIET REQUIRED)

add_subdirectory(src)
