##
# @file CMakeLists.txt
# 
# @copyright Copyright (C) 2020 srcML, LLC. (www.srcML.org)
# 
# The srcML Toolkit is free software; you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation; either version 2 of the License, or
# (at your option) any later version.
# 
# The srcML Toolkit is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
# 
# You should have received a copy of the GNU General Public License
# along with the srcML Toolkit; if not, write to the Free Software
# Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA
# 
# Build ANTLR 2 static library

cmake_minimum_required(VERSION 3.19)

project(srcML-LibAntlr)

# Download cpp library parts of antlr 2.7.7
set(ANTLR2_DIRNAME antlr-2.7.7)
set(ANTLR2_ARCHIVE ${ANTLR2_DIRNAME}.tar.gz)
set(ANTLR2_URL https://www.antlr2.org/download/${ANTLR2_ARCHIVE})
file(DOWNLOAD ${ANTLR2_URL} ${ANTLR2_ARCHIVE})

# Setup Universal architecture for Big Sur default
string(SUBSTRING ${CMAKE_SYSTEM_VERSION} 0 2 OS_VERSION)
if (OS_VERSION GREATER_EQUAL "20")
    if(NOT CMAKE_OSX_ARCHITECTURES)
        set(CMAKE_OSX_ARCHITECTURES "x86_64;arm64")
    endif()
    message(STATUS "OSX Architectures: ${CMAKE_OSX_ARCHITECTURES}")
endif()

# Extract the include and cpp files
file(ARCHIVE_EXTRACT INPUT ${ANTLR2_ARCHIVE}
    PATTERNS ${ANTLR2_DIRNAME}/lib/cpp/src/*.cpp ${ANTLR2_DIRNAME}/lib/cpp/antlr/*.hpp
)
set(ANTLR2_CPP_DIR ${CMAKE_CURRENT_BINARY_DIR}/${ANTLR2_DIRNAME}/lib/cpp)
file(REMOVE ${ANTLR2_CPP_DIR}/src/dll.cpp)

# Include files in the source
include_directories(${ANTLR2_CPP_DIR}/)

# Install include files
install(DIRECTORY ${ANTLR2_CPP_DIR}/antlr DESTINATION /usr/local/include)

# Source files
file(GLOB LIBSRC ${ANTLR2_CPP_DIR}/src/*.cpp)

# ANTLR2 static library
add_library(antlr ${LIBSRC})
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/libantlr.a DESTINATION /usr/local/lib)

# Display macOS Big Sur architectures of libantlr
if (OS_VERSION GREATER_EQUAL "20")
    add_custom_command(TARGET antlr POST_BUILD
                       COMMAND ${CMAKE_COMMAND} -E echo_append "-- OSX Architectures libantlr.a: "
                       COMMAND lipo -archs ${CMAKE_CURRENT_BINARY_DIR}/libantlr.a
    )
endif()
