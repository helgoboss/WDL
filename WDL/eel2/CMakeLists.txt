# Support CMake version shipping with Ubuntu 18.04 LTS
cmake_minimum_required(VERSION 3.10)

project(wdl-eel2 LANGUAGES C CXX)
option(NOOP_MUTEXES "Implements NSEEL_HOSTSTUB_EnterMutex and NSEEL_HOSTSTUB_LeaveMutex as no-op" OFF)

# Main target
add_library(wdl-eel2
    nseel-cfunc.c
    nseel-compiler.c
    nseel-caltab.c
    nseel-eval.c
    nseel-lextab.c
    nseel-ram.c
    nseel-yylex.c
    # This is just for defining NSEEL_HOSTSTUB_EnterMutex and NSEEL_HOSTSTUB_LeaveMutex as no-op if option set
    $<$<BOOL:${NOOP_MUTEXES}>:loose_eel.cpp>
    )
# Use generator syntax for INTERFACE-scoped includes to support usage of installed library (typical "Modern CMake"
# pattern, see https://pabloariasal.github.io/2018/02/19/its-time-to-do-cmake-right/)
target_include_directories(wdl-eel2
    PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>/..
    $<INSTALL_INTERFACE:include>
    )
if (MSVC)
  if (NOT ${MSVC_C_ARCHITECTURE_ID} MATCHES X86)
    target_link_libraries(wdl-eel2 PRIVATE asm-nseel-x64.obj)
  endif ()
elseif (APPLE)
  target_link_libraries(wdl-eel2 PRIVATE eel2/asm-nseel-x64-macho.o)
endif ()
add_library(wdl-eel2::wdl-eel2 ALIAS wdl-eel2)

# Install
include(GNUInstallDirs)
install(FILES ns-eel.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/eel2)
install(FILES ../wdltypes.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(TARGETS wdl-eel2
    EXPORT wdl-eel2-config
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    )
install(EXPORT wdl-eel2-config
    NAMESPACE wdl-eel2::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/wdl-eel2
    )