cmake_minimum_required(VERSION 3.5)
project(tests-pisa)

if (NOT DEFINED SS_ENDIAN)
    message(FATAL_ERROR "SS_ENDIAN is not defined")
endif()
