# Copyright (c) 2007-2014, Lloyd Hilaiel <me@lloyd.io>
#
# Permission to use, copy, modify, and/or distribute this software for any
# purpose with or without fee is hereby granted, provided that the above
# copyright notice and this permission notice appear in all copies.
#
# THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
# WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
# MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
# ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
# WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
# ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
# OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.

include(GNUInstallDirs)

set(config_install_dir "${CMAKE_INSTALL_LIBDIR}/cmake/YAJL")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")

set(version_config "${generated_dir}/YAJLConfigVersion.cmake")
set(project_config "${generated_dir}/YAJLConfig.cmake")
set(TARGETS_EXPORT_NAME "YAJLTargets")

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

configure_package_config_file(
    "cmake/Config.cmake.in"
    "${project_config}"
    INSTALL_DESTINATION "${config_install_dir}"
)

SET (SRCS yajl.c yajl_lex.c yajl_parser.c yajl_buf.c
          yajl_encode.c yajl_gen.c yajl_alloc.c
          yajl_tree.c yajl_version.c
)
SET (HDRS yajl_parser.h yajl_lex.h yajl_buf.h yajl_encode.h yajl_alloc.h)
SET (PUB_HDRS api/yajl_parse.h api/yajl_gen.h api/yajl_common.h api/yajl_tree.h)

# useful when fixing lexer bugs.
#ADD_DEFINITIONS(-DYAJL_LEXER_DEBUG)

# Ensure defined when building YAJL (as opposed to using it from
# another project).  Used to ensure correct function export when
# building win32 DLL.
ADD_DEFINITIONS(-DYAJL_BUILD)

# set up some paths
SET (libDir ${CMAKE_CURRENT_BINARY_DIR}/../${YAJL_DIST_NAME}/lib)
SET (incDir ${CMAKE_CURRENT_BINARY_DIR}/../${YAJL_DIST_NAME}/include/yajl)
SET (shareDir ${CMAKE_CURRENT_BINARY_DIR}/../${YAJL_DIST_NAME}/share/pkgconfig)

# set the output path for libraries
SET(LIBRARY_OUTPUT_PATH ${libDir})

ADD_LIBRARY(yajl ${SRCS} ${HDRS} ${PUB_HDRS})

#### setup shared library version number
SET_TARGET_PROPERTIES(yajl PROPERTIES
                      DEFINE_SYMBOL YAJL_SHARED
                      SOVERSION ${YAJL_MAJOR}
                      VERSION ${YAJL_MAJOR}.${YAJL_MINOR}.${YAJL_MICRO})

#### ensure a .dylib has correct absolute installation paths upon installation
IF(APPLE)
  MESSAGE("INSTALL_NAME_DIR: ${CMAKE_INSTALL_PREFIX}/lib")
  SET_TARGET_PROPERTIES(yajl PROPERTIES
                        INSTALL_NAME_DIR "${CMAKE_INSTALL_PREFIX}/lib")
ENDIF(APPLE)

#### build up an sdk as a post build step

# create some directories
FILE(MAKE_DIRECTORY ${libDir})
FILE(MAKE_DIRECTORY ${incDir})

# generate build-time source
SET(dollar $)
CONFIGURE_FILE(api/yajl_version.h.cmake ${incDir}/yajl_version.h)
CONFIGURE_FILE(yajl.pc.cmake ${shareDir}/yajl.pc)

# copy public headers to output directory
FOREACH (header ${PUB_HDRS})
  SET (header ${CMAKE_CURRENT_SOURCE_DIR}/${header})

  EXEC_PROGRAM(${CMAKE_COMMAND} ARGS -E copy_if_different ${header} ${incDir})
ENDFOREACH (header ${PUB_HDRS})

INCLUDE_DIRECTORIES(${incDir}/..)

INSTALL(TARGETS yajl
        EXPORT "${TARGETS_EXPORT_NAME}"
        RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
        LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
        ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
        INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")
INSTALL(FILES ${PUB_HDRS} DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/yajl")
INSTALL(FILES ${incDir}/yajl_version.h DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/yajl")
INSTALL(FILES ${shareDir}/yajl.pc DESTINATION "${CMAKE_INSTALL_DATAROOTDIR}/pkgconfig")

install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${TARGETS_EXPORT_NAME}"
    NAMESPACE "YAJL::"
    DESTINATION "${config_install_dir}"
)
