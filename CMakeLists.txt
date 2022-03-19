# Copyright (C) 2021, Xilinx Inc - All rights reserved
# Xilinx Resource Manger U30 Encoder Plugin 
                                    
# Licensed under the Apache License, Version 2.0 (the "License"). You may
# not use this file except in compliance with the License. A copy of the
# License is located at

#     http://www.apache.org/licenses/LICENSE-2.0

# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
# WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
# License for the specific language governing permissions and limitations 
# under the License.

cmake_minimum_required(VERSION 2.8.11)
include (CMakeDependentOption)
set(XRMENCODER_PROJ "xilinx-u30-xrm-encoder")
set(XRMENCODER_PROJ_VERSION "1.1.0")

find_program(UNAME_EXEC uname)
find_program(LSB_RELEASE_EXEC lsb_release)

# Grab Arch information like x86-64
execute_process(COMMAND ${UNAME_EXEC} -p
    OUTPUT_VARIABLE UNAME_ARCHTYPE
    OUTPUT_STRIP_TRAILING_WHITESPACE)
# Grab OS release short name
execute_process(COMMAND ${LSB_RELEASE_EXEC} -rs
	OUTPUT_VARIABLE LSB_RELEASE_REL_SHORT
	OUTPUT_STRIP_TRAILING_WHITESPACE)

set(XRMENCODER_PROJ_NAME "${XRMENCODER_PROJ}_${XRMENCODER_PROJ_VERSION}_${LSB_RELEASE_REL_SHORT}-${UNAME_ARCHTYPE}")

project(${XRMENCODER_PROJ})
set(ENV{PKG_CONFIG_PATH} "$ENV{PKG_CONFIG_PATH}:/usr/lib/pkgconfig")

# Linking variables
find_package(PkgConfig REQUIRED)
pkg_check_modules(XMA REQUIRED libxma2api)
pkg_check_modules(LIBXRM REQUIRED libxrm)

set (CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} --stats -Bsymbolic")

set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_CXX_FLAGS	"${CMAKE_CXX_FLAGS} -std=c++11 -Wall -Wextra -DDSA64 -fpermissive")

# Encoder Plugin
add_library(xrmU30Encoder SHARED
	codec-encoder-xrm-plg-u30.cpp
)

include_directories(.)

target_compile_options(xrmU30Encoder
	PUBLIC	${XMA_CFLAGS}
        PUBLIC ${LIBXRM_CFLAGS}
)

target_include_directories(xrmU30Encoder
        PUBLIC ${LIBXRM_INCLUDE_DIRS}
)

# Set the location for library installation
install(TARGETS xrmU30Encoder DESTINATION ${CMAKE_INSTALL_PREFIX}/plugin)

# Packaging section
set(CPACK_PACKAGE_NAME "${XRMENCODER_PROJ}")
set(CPACK_PACKAGE_FILE_NAME "${XRMENCODER_PROJ_NAME}")
set(CPACK_PACKAGE_VERSION "${XRMENCODER_PROJ_VERSION}")
set(CPACK_PACKAGE_CONTACT "Rohit Athavale <rohit.athavale@xilinx.com>")
set(CPACK_PACKAGE_VENDOR "Xilinx")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Xilinx MPSoC XRM Encoder Plugin")
set(CPACK_PACKAGE_DESCRIPTION "The Xilinx MPSoC based XRM encoder plugin")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "xrm (>= 1.3) ")
set(CPACK_RPM_PACKAGE_DEPENDS "xrm (>= 1.3) ")
set(CPACK_RPM_EXCLUDE_FROM_AUTO_FILELIST_ADDITION "/opt" "/opt/xilinx" "/usr/lib" "/usr/lib/pkgconfig")
include(CPack)
