cmake_minimum_required(VERSION 3.5)
project(SimpleScalar C)

option(USE_PISA "Use config-pisa" ON)
option(USE_PISA_BIG "Use config-pisabig" OFF)
option(USE_PISA_LITTLE "Use config-pisalit" OFF)
option(USE_ALPHA "Use config-alpha" OFF)

macro(choose_one)
    set(_turned_on 0)
    foreach(src ${ARGN})
        if (${src})
            message("${src} in ON")
            MATH(EXPR _turned_on "${_turned_on} + 1")
        endif()
    endforeach()
    message("_turned_on: ${_turned_on}")
    if (${_turned_on} EQUAL 0)
        message(FATAL_ERROR "1 option must be enabled from '${ARGN}'")
    elseif(${_turned_on} GREATER 1)
        message(FATAL_ERROR "Only 1 option can be enabled from '${ARGN}'")
    endif()
endmacro()

choose_one(USE_PISA USE_PISA_BIG USE_PISA_LITTLE USE_ALPHA)


if (UNIX)

elseif (WIN32)

endif()

function(ss_create_symlink old new)
    set(SS_RESULT 1)
    if(EXISTS ${old})
        if (WIN32)
            message(FATAL_ERROR "")
            if((${CMAKE_VERSION} VERSION_EQUAL "3.13") OR (${CMAKE_VERSION} VERSION_GREATER "3.13"))
                execute_process(
                        COMMAND ${CMAKE_COMMAND} -E create_symlink ${old} ${new}
                        RESULT_VARIABLE SS_RESULT
                        ERROR_QUIET)
            endif()

            if (${CMAKE_VERSION} VERSION_LESS "3.13" OR SS_RESULT EQUAL 1)
                get_filename_component(real_path "${new}" REALPATH)
                string(REPLACE "/" "\\" new_path "${real_path}")
                get_filename_component(real_path "${old}" REALPATH)
                string(REPLACE "/" "\\" old_path "${real_path}")
                execute_process(
                        COMMAND cmd /C mklink /J "${new_path}" "${old_path}"
                        RESULT_VARIABLE SS_RESULT
                        ERROR_QUIET)
            endif()
        else()
            execute_process(
                    COMMAND ${CMAKE_COMMAND} -E create_symlink ${old} ${new}
                    RESULT_VARIABLE SS_RESULT
                    ERROR_QUIET)

            if (SS_RESULT EQUAL 1)
                execute_process(
                        COMMAND ln -s ${old} ${new}
                        RESULT_VARIABLE SS_RESULT
                        ERROR_QUIET)
            endif()
        endif()
    endif()
    if (SS_RESULT EQUAL 0)
        message("Create symlink ${new} -> ${old}")
    else()
        message(WARNING "Failed to create symlink ${new} -> ${old}")
    endif()
endfunction()

macro(ss_run_probe RESULT)
    try_run(SS_RRV SS_CRV
            "${CMAKE_BINARY_DIR}/temp"
            "${CMAKE_CURRENT_LIST_DIR}/sysprobe.c"
            COMPILE_OUTPUT_VARIABLE SS_COV
            RUN_OUTPUT_VARIABLE ${RESULT}
            ARGS "${ARGN}"
            )
    if (NOT SS_COV)
        message(FATAL_ERROR "Failed to compile sysprobe.c")
    endif()
endmacro()

macro(ss_endian RESULT)
    ss_run_probe(${RESULT} "-s")
endmacro()

macro(ss_mflags RESULT)
    ss_run_probe(${RESULT} "-flags")
endmacro()

macro(ss_mlibs RESULT)
    ss_run_probe(_temp "-libs")
    if ("${_temp}" STREQUAL " \n")
        set(${RESULT} "-lm")
    else()
        set(${RESULT} "${_temp} -lm")
    endif()
endmacro()

set(ALPHA_DIR "${CMAKE_CURRENT_LIST_DIR}/target-alpha")
set(PISA_DIR "${CMAKE_CURRENT_LIST_DIR}/target-pisa")

set(GENERATED_DIR           "${CMAKE_CURRENT_LIST_DIR}/generated")
set(LINK_ALPHA_DIR          "${GENERATED_DIR}/alpha")
set(LINK_PISA_DIR           "${GENERATED_DIR}/pisa")
set(LINK_PISA_BIG_DIR       "${GENERATED_DIR}/pisa-big")
set(LINK_PISA_LITTLE_DIR    "${GENERATED_DIR}/pisa-little")

file(MAKE_DIRECTORY ${GENERATED_DIR})

if (USE_PISA)
    # Create symlinks for pisa
    file(MAKE_DIRECTORY ${LINK_PISA_DIR})
    ss_create_symlink(${PISA_DIR}/config.h  ${LINK_PISA_DIR}/config.h)
    ss_create_symlink(${PISA_DIR}/pisa.h    ${LINK_PISA_DIR}/machine.h)
    ss_create_symlink(${PISA_DIR}/pisa.c    ${LINK_PISA_DIR}/machine.c)
    ss_create_symlink(${PISA_DIR}/pisa.def  ${LINK_PISA_DIR}/machine.def)
    ss_create_symlink(${PISA_DIR}/loader.c  ${LINK_PISA_DIR}/loader.c)
    ss_create_symlink(${PISA_DIR}/symbol.c  ${LINK_PISA_DIR}/symbol.c)
    ss_create_symlink(${PISA_DIR}/syscall.c ${LINK_PISA_DIR}/syscall.c)
    set(LINK_DIR "${LINK_PISA_DIR}")
elseif (USE_PISA_BIG)
    # Create symlinks for pisa-big
    file(MAKE_DIRECTORY ${LINK_PISA_BIG_DIR})
    ss_create_symlink(${PISA_DIR}/configbig.h   ${LINK_PISA_BIG_DIR}/config.h)
    ss_create_symlink(${PISA_DIR}/pisa.h        ${LINK_PISA_BIG_DIR}/machine.h)
    ss_create_symlink(${PISA_DIR}/pisa.c        ${LINK_PISA_BIG_DIR}/machine.c)
    ss_create_symlink(${PISA_DIR}/pisa.def      ${LINK_PISA_BIG_DIR}/machine.def)
    ss_create_symlink(${PISA_DIR}/loader.c      ${LINK_PISA_BIG_DIR}/loader.c)
    ss_create_symlink(${PISA_DIR}/symbol.c      ${LINK_PISA_BIG_DIR}/symbol.c)
    ss_create_symlink(${PISA_DIR}/syscall.c     ${LINK_PISA_BIG_DIR}/syscall.c)
    set(LINK_DIR "${LINK_PISA_BIG_DIR}")
elseif (USE_PISA_LITTLE)
    # Create symlinks for pisa-little
    file(MAKE_DIRECTORY ${LINK_PISA_LITTLE_DIR})
    ss_create_symlink(${PISA_DIR}/configlit.h   ${LINK_PISA_LITTLE_DIR}/config.h)
    ss_create_symlink(${PISA_DIR}/pisa.h        ${LINK_PISA_LITTLE_DIR}/machine.h)
    ss_create_symlink(${PISA_DIR}/pisa.c        ${LINK_PISA_LITTLE_DIR}/machine.c)
    ss_create_symlink(${PISA_DIR}/pisa.def      ${LINK_PISA_LITTLE_DIR}/machine.def)
    ss_create_symlink(${PISA_DIR}/loader.c      ${LINK_PISA_LITTLE_DIR}/loader.c)
    ss_create_symlink(${PISA_DIR}/symbol.c      ${LINK_PISA_LITTLE_DIR}/symbol.c)
    ss_create_symlink(${PISA_DIR}/syscall.c     ${LINK_PISA_LITTLE_DIR}/syscall.c)
    set(LINK_DIR "${LINK_PISA_LITTLE_DIR}")
elseif (USE_ALPHA)
    # Create symlinks for alpha
    file(MAKE_DIRECTORY ${LINK_ALPHA_DIR})
    ss_create_symlink(${ALPHA_DIR}/config.h     ${LINK_ALPHA_DIR}/config.h)
    ss_create_symlink(${ALPHA_DIR}/alpha.h      ${LINK_ALPHA_DIR}/machine.h)
    ss_create_symlink(${ALPHA_DIR}/alpha.c      ${LINK_ALPHA_DIR}/machine.c)
    ss_create_symlink(${ALPHA_DIR}/alpha.def    ${LINK_ALPHA_DIR}/machine.def)
    ss_create_symlink(${ALPHA_DIR}/loader.c     ${LINK_ALPHA_DIR}/loader.c)
    ss_create_symlink(${ALPHA_DIR}/symbol.c     ${LINK_ALPHA_DIR}/symbol.c)
    ss_create_symlink(${ALPHA_DIR}/syscall.c    ${LINK_ALPHA_DIR}/syscall.c)
    set(LINK_DIR "${LINK_ALPHA_DIR}")
endif()

set(exo_bait_dir "${LINK_DIR}/compat_depth")
file(MAKE_DIRECTORY "${exo_bait_dir}")

include_directories(
        "${exo_bait_dir}"
        "${CMAKE_CURRENT_SOURCE_DIR}")

ss_mflags(MFLAGS)
ss_mlibs(MLIBS)
message("MFLAGS: ${MFLAGS}")
message("MLIBS: ${MLIBS}")

set(FFLAGS "-DDEBUG")
set(OFLAGS "-O0 -g -Wall")
set(CFLAGS "${MFLAGS} ${FFLAGS} ${OFLAGS} ${BINUTILS_INC} ${BINUTILS_LIB}")
add_subdirectory(libexo)

set(OBJS
        main.c
        memory.c
        regs.c
        endian.c
        dlite.c
        eval.c
        options.c
        stats.c
        eio.c
        range.c
        misc.c
        ${LINK_DIR}/symbol.c
        ${LINK_DIR}/syscall.c
        ${LINK_DIR}/loader.c
        ${LINK_DIR}/machine.c
        )

macro(add_sim name src)
    add_executable(${name} ${src} "${ARGN}" ${OBJS})
    target_compile_options(${name} PUBLIC ${CFLAGS})
    target_link_libraries(${name} libexo "${MLIBS}")
    target_include_directories(${name} PUBLIC
            "${CMAKE_CURRENT_SOURCE_DIR}"
            "${LINK_DIR}")
    add_dependencies(${name} libexo)
endmacro()

add_sim(sim_fast sim-fast.c)
add_sim(sim_safe sim-safe.c)
add_sim(sim_profile sim-profile.c)
add_sim(sim_cache sim-cache.c cache.c)
add_sim(sim_bpred sim-bpred.c bpred.c)
add_sim(sim_outorder sim-outorder.c cache.c bpred.c resource.c ptrace.c)
