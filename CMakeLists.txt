cmake_minimum_required(VERSION 3.9)
project(bzip2 C)

add_library(bzip2.bz2 STATIC blocksort.c huffman.c crctable.c randtable.c compress.c decompress.c bzlib.c)
target_include_directories(bzip2.bz2 PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<INSTALL_INTERFACE:include/bzip2>)
target_compile_definitions(bzip2.bz2 PRIVATE "_FILE_OFFSET_BITS=64")
if(UNIX OR (WIN32 AND NOT CMAKE_SHARED_LIBRARY_PREFIX STREQUAL "")
        OR (WIN32 AND NOT CMAKE_STATIC_LIBRARY_PREFIX STREQUAL ""))
    set_target_properties(bzip2.bz2 PROPERTIES OUTPUT_NAME bz2)
else()
    set_target_properties(bzip2.bz2 PROPERTIES OUTPUT_NAME libbz2)
endif()
add_library(bzip2::bz2 ALIAS bzip2.bz2)
set_target_properties(bzip2.bz2 PROPERTIES EXPORT_NAME "bz2")

add_executable(bzip2.bzip2 bzip2.c)
target_link_libraries(bzip2.bzip2 PRIVATE bzip2.bz2)
set_target_properties(bzip2.bzip2 PROPERTIES OUTPUT_NAME bzip2)

add_executable(bzip2.bzip2recover bzip2recover.c)
target_link_libraries(bzip2.bzip2recover PRIVATE bzip2.bz2)
set_target_properties(bzip2.bzip2recover PROPERTIES OUTPUT_NAME bzip2recover)

install(
    FILES ${PROJECT_SOURCE_DIR}/bzlib.h ${PROJECT_SOURCE_DIR}/bzlib_private.h
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/bzip2)
install(
    TARGETS bzip2.bz2 bzip2.bzip2 bzip2.bzip2recover
    EXPORT bzip2
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    FRAMEWORK DESTINATION Library/Frameworks)
install(EXPORT bzip2
    NAMESPACE bzip2::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/bzip2
    FILE bzip2.cmake)
