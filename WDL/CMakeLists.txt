# Support CMake version shipping with Ubuntu 18.04 LTS
cmake_minimum_required(VERSION 3.10)
project(wdl LANGUAGES C CXX)
option(WDL_EEL2 "Builds eel2" OFF)

if (WDL_EEL2)
  # EEL2 target
  add_library(wdl-eel2
      eel2/nseel-cfunc.c
      eel2/nseel-compiler.c
      eel2/nseel-caltab.c
      eel2/nseel-eval.c
      eel2/nseel-lextab.c
      eel2/nseel-ram.c
      eel2/nseel-yylex.c
      )
  # Use generator syntax for INTERFACE-scoped includes to support usage of installed library (typical "Modern CMake"
  # pattern, see https://pabloariasal.github.io/2018/02/19/its-time-to-do-cmake-right/)
  target_include_directories(wdl-eel2
      PUBLIC
      $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
      $<INSTALL_INTERFACE:.>
      )
  if (MSVC)
    if (NOT ${MSVC_C_ARCHITECTURE_ID} MATCHES X86)
      target_link_libraries(wdl-eel2 PRIVATE ${PROJECT_SOURCE_DIR}/eel2/asm-nseel-x64.obj)
    endif ()
  elseif (APPLE)
    target_link_libraries(wdl-eel2 PRIVATE ${PROJECT_SOURCE_DIR}/eel2/asm-nseel-x64-macho.o)
  endif ()
  add_library(wdl-eel2::wdl-eel2 ALIAS wdl-eel2)

  # Install
  include(GNUInstallDirs)
  install(FILES eel2/ns-eel.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/eel2)
  install(FILES wdltypes.h DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
  install(TARGETS wdl-eel2
      EXPORT wdl-eel2-config
      LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
      ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
      )
  install(EXPORT wdl-eel2-config
      NAMESPACE wdl-eel2::
      DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/wdl-eel2
      )
endif ()