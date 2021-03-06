#-------------------------------------------------------------------------------
# Copyright (C) 2019-2020 Zilliz. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance
# with the License. You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software distributed under the License
# is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express
# or implied. See the License for the specific language governing permissions and limitations under the License.
#-------------------------------------------------------------------------------

if ( DEFINED ENV{MILVUS_SQLITE_URL} )
    set( SQLITE_SOURCE_URL "$ENV{MILVUS_SQLITE_URL}" )
else ()
    set( SQLITE_SOURCE_URL "https://www.sqlite.org/2019/sqlite-autoconf-${SQLITE_VERSION}.tar.gz" )
endif ()

macro( build_sqlite )
    message(STATUS "Building SQLITE-${SQLITE_VERSION} from source")

    ExternalProject_Add(
            sqlite_ep
            PREFIX              ${CMAKE_BINARY_DIR}/3rdparty_download/sqlite-subbuild
            DOWNLOAD_DIR        ${THIRDPARTY_DOWNLOAD_PATH}
            INSTALL_DIR         ${CMAKE_CURRENT_BINARY_DIR}
            URL                 ${SQLITE_SOURCE_URL}
            URL_MD5             "3c68eb400f8354605736cd55400e1572"
            CONFIGURE_COMMAND   <SOURCE_DIR>/configure
                                    "--prefix=<INSTALL_DIR>"
                                    "CC=${EP_C_COMPILER}"
                                    "CXX=${EP_CXX_COMPILER}"
                                    "CFLAGS=${EP_C_FLAGS}"
                                    "CXXFLAGS=-O3 -fPIC -Wno-error -fopenmp"
            BUILD_COMMAND       ${MAKE} ${MAKE_BUILD_ARGS}
            INSTALL_COMMAND     ${MAKE} install
            ${EP_LOG_OPTIONS} )

    ExternalProject_Get_Property( sqlite_ep INSTALL_DIR )
    if( NOT IS_DIRECTORY ${INSTALL_DIR}/include )
        file( MAKE_DIRECTORY "${INSTALL_DIR}/include" )
    endif()

    add_library(sqlite STATIC IMPORTED)
    set_target_properties( sqlite
            PROPERTIES
                IMPORTED_GLOBAL                 TRUE
                IMPORTED_LOCATION               ${INSTALL_DIR}/lib/libsqlite3.a
                INTERFACE_INCLUDE_DIRECTORIES   ${INSTALL_DIR}/include )
    add_dependencies(sqlite sqlite_ep)
endmacro()

build_sqlite()
