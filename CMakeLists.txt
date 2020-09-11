cmake_minimum_required(VERSION 3.3 FATAL_ERROR)
project(gslib C Fortran)

#===============================================================================
# RPATH and install information
#===============================================================================

# This block of code ensures that dynamic libraries can be found via the RPATH
# whether the executable is the original one from the build directory or the
# installed one in CMAKE_INSTALL_PREFIX. Ref:
# https://gitlab.kitware.com/cmake/community/wikis/doc/cmake/RPATH-handling

# use, i.e. don't skip the full RPATH for the build tree
set(CMAKE_SKIP_BUILD_RPATH  FALSE)

# when building, don't use the install RPATH already
# (but later on when installing)
set(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)

set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")

# add the automatically determined parts of the RPATH
# which point to directories outside the build tree to the install RPATH
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

# =============================================================================
# Library target
# =============================================================================

set(gs_sources
    src/comm.c
    src/crystal.c
    src/fail.c
    src/gs.c
    src/gs_local.c
    src/sarray_sort.c
    src/sarray_transfer.c
    src/sort.c
    src/tensor.c)

set(fwrapper_sources
    src/fcrystal.c
    src/findpts.c)

set(intp_sources
    src/findpts_el_2.c
    src/findpts_el_3.c
    src/findpts_local.c
    src/lob_bnd.c
    src/obbox.c
    src/poly.c)

add_library(libgs
    ${gs_sources}
    ${fwrapper_sources}
    ${intp_sources})
set_target_properties(libgs PROPERTIES OUTPUT_NAME gs)
target_include_directories(libgs PUBLIC src ${CMAKE_CURRENT_BINARY_DIR})
target_link_libraries(libgs PUBLIC m)

# =============================================================================
# Test targets
# =============================================================================

foreach(test
    comm_test
    crystal_test
    findpts_el_2_test
    findpts_el_2_test2
    findpts_el_3_test
    findpts_el_3_test2
    findpts_local_test
    findpts_test
    gs_test
    gs_test_gop_blocking
    gs_test_gop_nonblocking
    gs_unique_test
    lob_bnd_test
    obbox_test
    poly_test
    sarray_sort_test
    sarray_transfer_test
    sort_test
    sort_test2)
  add_executable(${test} tests/${test}.c)
  target_include_directories(${test} PUBLIC src tests ${CMAKE_CURRENT_BINARY_DIR})
  target_link_libraries(${test} PUBLIC libgs m)
endforeach()

add_executable(f-igs tests/fortran/f-igs.f)
target_include_directories(f-igs PUBLIC src tests ${CMAKE_CURRENT_BINARY_DIR})
target_link_libraries(f-igs PUBLIC libgs)

# =============================================================================
# Preprocessor Symbols
# =============================================================================

# MPI
if(MPI OR NOT DEFINED MPI)
  set(MPI TRUE)
  target_compile_definitions(libgs PRIVATE -DMPI)
endif()

# UNDERSCORE
include(FortranCInterface)
FortranCInterface_VERIFY()
if(FortranCInterface_GLOBAL_SUFFIX STREQUAL "_")
  set(UNDERSCORE TRUE)
  target_compile_definitions(libgs PRIVATE -DUNDERSCORE)
endif()
set(CMAKE_Fortran_FORMAT "FIXED")

# GLOBAL_LONG_LONG
set(GLOBAL_LONG_LONG TRUE)
target_compile_definitions(libgs PRIVATE -DGLOBAL_LONG_LONG)

# PREFIX
if(NOT DEFINED PREFIX)
  set(PREFIX gslib_)
endif()
target_compile_definitions(libgs PRIVATE -DPREFIX=${PREFIX})

# FPREFIX
if(NOT DEFINED FPREFIX)
  set(FPREFIX fgslib_)
endif()
target_compile_definitions(libgs PRIVATE -DFPREFIX=${FPREFIX})

# USREXIT
if(USREXIT)
  target_compile_definitions(libgs PRIVATE -DUSE_USR_EXIT)
endif()

# USREXIT
if(NBC)
  target_compile_definitions(libgs PRIVATE -DUSE_NBC)
endif()

# BLAS
if(BLAS)
  target_compile_definitions(libgs PRIVATE -DUSE_CBLAS)
else()
  target_compile_definitions(libgs PRIVATE -DUSE_NAIVE_BLAS)
  set(USE_NAIVE_BLAS TRUE)
endif()

# Debug
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
  target_compile_definitions(libgs PRIVATE -DGSLIB_DEBUG)
endif()

configure_file(config/config.h.in config.h @ONLY)

#################################################################################
# Install targets
#################################################################################

install(TARGETS 
  libgs
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
)
