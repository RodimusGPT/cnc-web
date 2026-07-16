cmake_minimum_required(VERSION 3.25)

project(CncWeb LANGUAGES C CXX)

option(BUILD_WEBTD "Build the Tiberian Dawn WebAssembly module." OFF)
option(BUILD_TESTS "Build the browser adapter tests." ON)
set(
    VANILLA_CONQUER_SOURCE_DIR
    "${CMAKE_CURRENT_SOURCE_DIR}/vendor/vanilla-conquer"
    CACHE PATH
    "Path to the pinned Vanilla-Conquer source tree."
)

if(NOT EXISTS "${VANILLA_CONQUER_SOURCE_DIR}/tiberiandawn/dllinterface.cpp")
    message(FATAL_ERROR
        "The Vanilla-Conquer submodule is missing. "
        "Run: git submodule update --init --recursive")
endif()

if(BUILD_TESTS)
    enable_testing()
    add_custom_target(tests)
endif()

add_subdirectory(browser/engine)

if(BUILD_WEBTD)
    if(NOT EMSCRIPTEN)
        message(FATAL_ERROR "BUILD_WEBTD requires the Emscripten toolchain.")
    endif()

    set(BUILD_REMASTERTD_CORE ON CACHE BOOL "" FORCE)
    set(BUILD_REMASTERTD OFF CACHE BOOL "" FORCE)
    set(BUILD_REMASTERRA OFF CACHE BOOL "" FORCE)
    set(BUILD_VANILLATD OFF CACHE BOOL "" FORCE)
    set(BUILD_VANILLARA OFF CACHE BOOL "" FORCE)
    set(BUILD_TESTS OFF CACHE BOOL "" FORCE)
    set(BUILD_TOOLS OFF CACHE BOOL "" FORCE)
    set(MAP_EDITORTD OFF CACHE BOOL "" FORCE)
    set(MAP_EDITORRA OFF CACHE BOOL "" FORCE)
    set(NETWORKING OFF CACHE BOOL "" FORCE)
    set(SDL1 OFF CACHE BOOL "" FORCE)
    set(SDL2 OFF CACHE BOOL "" FORCE)
    set(OPENAL OFF CACHE BOOL "" FORCE)
    set(DSOUND OFF CACHE BOOL "" FORCE)
    set(DDRAW OFF CACHE BOOL "" FORCE)
    set(
        REMASTER_KEYBOARD_SOURCE
        "${CMAKE_CURRENT_SOURCE_DIR}/browser/engine/src/wwkeyboard_web.cpp"
        CACHE FILEPATH
        "Web keyboard backend for the remaster core."
        FORCE
    )

    add_subdirectory(
        "${VANILLA_CONQUER_SOURCE_DIR}"
        "${CMAKE_BINARY_DIR}/vanilla-conquer"
        EXCLUDE_FROM_ALL
    )

    set(_cnc_web_compat
        "${CMAKE_CURRENT_SOURCE_DIR}/browser/engine/src/td_platform_compat.h")
    set(_cnc_web_clock
        "${CMAKE_CURRENT_SOURCE_DIR}/browser/engine/src/td_clock_compat.h")

    target_compile_options(common PRIVATE -include "${_cnc_web_compat}")
    target_compile_options(commonr PRIVATE -include "${_cnc_web_compat}")
    set_property(
        SOURCE timer.cpp
        TARGET_DIRECTORY common
        APPEND
        PROPERTY COMPILE_OPTIONS -include "${_cnc_web_clock}"
    )

    target_compile_definitions(TiberianDawnRemasterCore PRIVATE CNC_WEB_BUILD)
    target_compile_options(TiberianDawnRemasterCore PRIVATE
        -fexceptions
        -fms-extensions
        -fsigned-char
        -fwrapv
        -fno-fast-math
        -ffp-contract=off
        -include "${_cnc_web_compat}"
        $<$<CONFIG:Release>:-g0>
    )

    add_executable(TiberianDawnWeb
        browser/engine/src/determinism.cpp
        browser/engine/src/td_backend.cpp
    )
    target_include_directories(TiberianDawnWeb PRIVATE
        browser/engine/include
        browser/engine/src
    )
    target_compile_definitions(TiberianDawnWeb PRIVATE
        TRUE_FALSE_DEFINED
        ENGLISH
    )
    target_compile_options(TiberianDawnWeb PRIVATE
        -fexceptions
        -fms-extensions
        -fsigned-char
        -fwrapv
        -fno-fast-math
        -ffp-contract=off
        -include "${_cnc_web_compat}"
        $<$<CONFIG:Release>:-g0>
    )
    target_link_libraries(TiberianDawnWeb PRIVATE
        cnc_web_adapter
        VanillaConquer::TiberianDawnRemasterCore
    )
    target_link_options(TiberianDawnWeb PRIVATE
        --no-entry
        -fexceptions
        -sDISABLE_EXCEPTION_CATCHING=0
        -sALLOW_MEMORY_GROWTH=1
        -sGROWABLE_ARRAYBUFFERS=0
        -sINITIAL_MEMORY=67108864
        -sMAXIMUM_MEMORY=402653184
        -sFILESYSTEM=1
        -sFORCE_FILESYSTEM=1
        -sENVIRONMENT=worker
        -sMODULARIZE=1
        -sEXPORT_ES6=1
        -sEXPORT_NAME=createTiberianDawnModule
        -sEXIT_RUNTIME=0
        -sWASM_BIGINT=1
        "SHELL:-sEXPORTED_RUNTIME_METHODS=['FS','HEAPU8','HEAPU32']"
        "SHELL:-sEXPORTED_FUNCTIONS=['_malloc','_free','_cnc_web_abi_version','_cnc_web_create','_cnc_web_destroy','_cnc_web_set_campaign_transition','_cnc_web_start','_cnc_web_submit_commands','_cnc_web_advance','_cnc_web_snapshot_size','_cnc_web_write_snapshot','_cnc_web_event_size','_cnc_web_poll_event','_cnc_web_save_size','_cnc_web_write_save','_cnc_web_load_save','_cnc_web_state_hash','_CNC_Web_Acceptance_Force_Victory']"
        -lworkerfs.js
        $<$<CONFIG:Release>:-g0>
    )
    set_target_properties(TiberianDawnWeb PROPERTIES
        OUTPUT_NAME "tiberiandawn"
        SUFFIX ".js"
    )
endif()
