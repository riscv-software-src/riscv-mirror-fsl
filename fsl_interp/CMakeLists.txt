cmake_minimum_required(VERSION 3.10)

project(fsl_interp)

find_package(Boost REQUIRED COMPONENTS program_options)
find_package(BISON REQUIRED)
find_package(FLEX REQUIRED)

set(BISON_SRC "${CMAKE_CURRENT_SOURCE_DIR}/src/fsl.y")
set(BISON_OUT "${CMAKE_CURRENT_BINARY_DIR}/yy.tab.cpp")
set(BISON_H "${CMAKE_CURRENT_BINARY_DIR}/yy.tab.hpp")
set(FLEX_SRC "${CMAKE_CURRENT_SOURCE_DIR}/src/fsl.l")
set(FLEX_OUT "${CMAKE_CURRENT_BINARY_DIR}/lex.yy.cpp")

BISON_TARGET(Parser "${BISON_SRC}" "${BISON_OUT}" DEFINES_FILE "${BISON_H}")
FLEX_TARGET(Lexer "${FLEX_SRC}" "${FLEX_OUT}")
ADD_FLEX_BISON_DEPENDENCY(Lexer Parser)

add_library(fslinterp STATIC
    src/FslParser.cpp
    src/Options.cpp
    "${BISON_OUT}"
    "${FLEX_OUT}"
)
set_target_properties(fslinterp PROPERTIES
    OUTPUT_NAME fsl_interp
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
)
target_include_directories(fslinterp
    PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/inc"
    PRIVATE "${CMAKE_CURRENT_BINARY_DIR}"
)
target_link_libraries(fslinterp PUBLIC Boost::program_options)
target_compile_options(fslinterp PRIVATE
    $<$<CXX_COMPILER_ID:Clang>:-Wno-error=unneeded-internal-declaration>
)

add_executable(fsl_interp src/main.cpp)
set_target_properties(fsl_interp PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
)
target_link_libraries(fsl_interp PRIVATE fslinterp)
