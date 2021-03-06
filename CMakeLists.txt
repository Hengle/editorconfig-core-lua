#
# Copyright (c) 2016 João Valverde <joao.valverde@tecnico.ulisboa.pt>
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions are met:
#
# 1. Redistributions of source code must retain the above copyright notice,
#    this list of conditions and the following disclaimer.
# 2. Redistributions in binary form must reproduce the above copyright notice,
#    this list of conditions and the following disclaimer in the documentation
#    and/or other materials provided with the distribution.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
# AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
# IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
# ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE
# LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR
# CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF
# SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS
# INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN
# CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
# ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE
# POSSIBILITY OF SUCH DAMAGE.
#

cmake_minimum_required(VERSION 3.0)

project(editorconfig-core-lua VERSION 0.3.0 LANGUAGES C)

set(LEC_VERSION "${PROJECT_VERSION}-git")

set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/Modules")
include(FindEditorConfig)

find_package(EditorConfig REQUIRED)
set(Lua_FIND_VERSION 5.2) # minimum Lua version
find_package(Lua REQUIRED)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRE ON)

add_library(editorconfig_core MODULE editorconfig_lua.c)
target_compile_definitions(editorconfig_core
    PRIVATE "LEC_VERSION=\"${LEC_VERSION}\""
)
target_include_directories(editorconfig_core SYSTEM
    PRIVATE ${EDITORCONFIG_INCLUDE_DIRS} ${LUA_INCLUDE_DIR}
)
target_link_libraries(editorconfig_core
    ${EDITORCONFIG_LIBRARIES} ${LUA_LIBRARIES}
)

set(ECL_LIBDIR "lib/lua/${LUA_VERSION_MAJOR}.${LUA_VERSION_MINOR}"
    CACHE PATH "Target installation directory."
)

install(TARGETS editorconfig_core
    DESTINATION ${ECL_LIBDIR}
)

configure_file(editorconfig.lua.in editorconfig.lua)

# Testing. Type "make test" to run tests. Only if the test submodule is
# checkouted should we do this
if(EXISTS ${CMAKE_SOURCE_DIR}/tests/CMakeLists.txt)
    enable_testing()
    find_program(LUA_CMD lua${LUA_VERSION_MAJOR}.${LUA_VERSION_MINOR})
    set(EDITORCONFIG_CMD ${LUA_CMD} ${CMAKE_BINARY_DIR}/editorconfig.lua)
    add_subdirectory(tests)
else()
    message(WARNING
        " Testing files are not found. Testing will not be available. If you obtained the source tree through git, please run `git submodule update --init` to update the tests submodule.")
endif()
