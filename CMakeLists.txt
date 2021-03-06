cmake_minimum_required(VERSION 2.8.12)
set(CMAKE_MACOSX_RPATH 1)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/Modules/")

if(MSVC AND NOT "${CMAKE_GENERATOR}" MATCHES "(Win64|IA64)")
  message(FATAL_ERROR "ERROR: Building with Visual Studio requires using a 64-bit project generator (e.g. cmake -G \"Visual Studio 15 2017 Win64\")")
endif()


find_package(Csound)

if(APPLE)
    set(PLUGIN_INSTALL_DIR "$ENV{HOME}/Library/Audio/Plug-Ins/ladspa")
else()
    set(PLUGIN_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/lib")
endif()

function(make_plugin_frontend libname srcs)
    if(APPLE)
        add_library(${libname} SHARED ${srcs})
    else()
        add_library(${libname} MODULE ${srcs})
    endif()

    set(i 2)
    while( ${i} LESS ${ARGC} )
        if(NOT MSVC OR NOT("${ARGV${i}}" MATCHES "m"))
            target_link_libraries(${libname} ${ARGV${i}})
        endif()
        math(EXPR i "${i}+1")
    endwhile()
    
   install(TARGETS ${libname} LIBRARY DESTINATION "${PLUGIN_INSTALL_DIR}" )
endfunction(make_plugin_frontend)


if(CSOUND_FOUND)
    message(STATUS "Building CsLadspa Plugin!")
    make_plugin_frontend(csladspa "src/csladspa.cpp" "${CSOUND_LIBRARY}")
    target_include_directories(csladspa PRIVATE ${CSOUND_INCLUDE_DIRS})

    if(MSVC) 
      target_include_directories(csladspa PRIVATE "${CMAKE_SOURCE_DIR}/include")
      target_link_libraries(csladspa PRIVATE ${CSOUND_LIBRARY})
    endif()

    # We do not want the lib prefix
    set_target_properties(csladspa PROPERTIES PREFIX "")
else()
    message(FATAL_ERROR "Csound library is required for CsLadspa")
endif()

