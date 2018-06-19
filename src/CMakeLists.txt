
set(JTHREAD_IMPORT "")
set(JTHREAD_EXPORT "")
if(MSVC AND NOT JTHREAD_COMPILE_STATIC)
    set(JTHREAD_IMPORT "__declspec(dllimport)")
    set(JTHREAD_EXPORT "__declspec(dllexport)")
endif()

configure_file("${CMAKE_CURRENT_SOURCE_DIR}/jthreadconfig.h.in"
               "${CMAKE_CURRENT_BINARY_DIR}/src/jthreadconfig.h"
               @ONLY)

set(HEADERS
    jmutex.h
    jthread.h
    jmutexautolock.h
    ${CMAKE_CURRENT_BINARY_DIR}/jthreadconfig.h)

if(CMAKE_USE_WIN32_THREADS_INIT)
    set(SOURCES win32/jmutex.cpp win32/jthread.cpp)
    set(JTHREAD_CONFIG_WIN32THREADS "#define JTHREAD_CONFIG_WIN32THREADS")
    set(JTHREAD_WIN32_CRITICALSECTION OFF CACHE BOOL "If set to false, use standard mutex. If set to true, use a critical section object.")
    if(JTHREAD_WIN32_CRITICALSECTION)
        set(JTHREAD_CONFIG_JMUTEXCRITICALSECTION "#define JTHREAD_CONFIG_JMUTEXCRITICALSECTION")
    else(JTHREAD_WIN32_CRITICALSECTION)
        set(JTHREAD_CONFIG_JMUTEXCRITICALSECTION "// Using standard Win32 mutex")
    endif(JTHREAD_WIN32_CRITICALSECTION)
else(CMAKE_USE_WIN32_THREADS_INIT) # Use pthread
    set(SOURCES pthread/jmutex.cpp pthread/jthread.cpp)
    set(JTHREAD_CONFIG_WIN32THREADS "// Using pthread based threads")
    set(JTHREAD_CONFIG_JMUTEXCRITICALSECTION "")
    check_library_exists(pthreads pthread_cancel "" JTHREAD_HAVE_PTHREADCANCEL)
    if(NOT JTHREAD_HAVE_PTHREADCANCEL)
        set(JTHREAD_HAVE_PTHREADCANCEL)
        check_library_exists(pthread pthread_cancel "" JTHREAD_HAVE_PTHREADCANCEL)
    endif()
endif(CMAKE_USE_WIN32_THREADS_INIT)

if(JTHREAD_COMPILE_STATIC)
    add_library(jthread STATIC ${SOURCES})
else(JTHREAD_COMPILE_STATIC)
    add_library(jthread SHARED ${SOURCES})
endif()

target_link_libraries(jthread PUBLIC Threads::Threads)

set_target_properties(jthread PROPERTIES
                      VERSION ${PROJECT_VERSION}
                      SOVERSION ${PROJECT_VERSION_MAJOR}
                      PUBLIC_HEADER "${HEADERS}")

target_compile_definitions(jthread PRIVATE JTHREAD_COMPILING)

target_include_directories(jthread PUBLIC
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
                           $<INSTALL_INTERFACE:include/>)

if(NOT JTHREAD_HAVE_PTHREADCANCEL)
    target_compile_definitions(jthread PRIVATE JTHREAD_SKIP_PTHREAD_CANCEL)
else()
#   message("pthread_cancel appears to exist")
endif(NOT JTHREAD_HAVE_PTHREADCANCEL)

install(FILES ${HEADERS} DESTINATION include/jthread)
install(TARGETS ${JTHREAD_INSTALLTARGETS} DESTINATION ${LIBRARY_INSTALL_DIR})

install(TARGETS jthread
        EXPORT jthread-export
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/jthread)

install(EXPORT jthread-export
        NAMESPACE jthread::
        CONFIGURATIONS Debug Release RelWithDebInfo MinSizeRel
        FILE jthread-config.cmake
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/jthread)