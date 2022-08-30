cmake_minimum_required(VERSION 3.5)
project(libexo)

add_custom_command(
        OUTPUT "${CMAKE_CURRENT_LIST_DIR}/exolex.c"
        COMMAND flex -o "${CMAKE_CURRENT_LIST_DIR}/exolex.c" "${CMAKE_CURRENT_LIST_DIR}/exolex.l"
        WORKING_DIRECTORY "${CMAKE_CURRENT_LIST_DIR}"
        DEPENDS exolex.l
)

add_library(exo
        "${CMAKE_CURRENT_LIST_DIR}/libexo.c"
        "${CMAKE_CURRENT_LIST_DIR}/exolex.c")

add_library(libexo ALIAS exo)
