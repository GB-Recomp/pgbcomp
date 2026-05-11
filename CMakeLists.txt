cmake_minimum_required(VERSION 3.16)
project(pgbcomp C CXX)

include(CheckIPOSupported)
include(FetchContent)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()
if(CMAKE_C_COMPILER_ID MATCHES "GNU|Clang")
    if(CMAKE_BUILD_TYPE STREQUAL "Debug")
        add_compile_options(-O0 -g)
    else()
        add_compile_options(-O3)
    endif()
endif()

# ---------------------------------------------------------------------------
# gb-recompiled runtime
# Fetched first so each per-game repo's `if(NOT TARGET gbrt)` skips its own
# fetch (they share this single gbrt target).
# ---------------------------------------------------------------------------
set(GBRT_REF "main" CACHE STRING "git ref of GB-Recomp/gb-recompiled to use")
FetchContent_Declare(gb_recompiled
    GIT_REPOSITORY https://github.com/GB-Recomp/gb-recompiled.git
    GIT_TAG ${GBRT_REF}
    GIT_SHALLOW TRUE
)
FetchContent_GetProperties(gb_recompiled)
if(NOT gb_recompiled_POPULATED)
    FetchContent_Populate(gb_recompiled)
endif()
add_subdirectory(${gb_recompiled_SOURCE_DIR}/runtime ${CMAKE_BINARY_DIR}/_gbrt_build)

# ---------------------------------------------------------------------------
# Per-game cart repos
# Each provides a <id>_cart static library; we just FetchContent them and
# link them all into the umbrella launcher below. Their standalone-launcher
# executables only build when they're the top-level CMake project, so
# nothing conflicts.
# ---------------------------------------------------------------------------
set(POKERED_REF      "main" CACHE STRING "git ref of GB-Recomp/pokered")
set(POKEBLUE_REF     "main" CACHE STRING "git ref of GB-Recomp/pokeblue")
set(POKEYELLOW_REF   "main" CACHE STRING "git ref of GB-Recomp/pokeyellow")
set(POKEGOLD_REF     "main" CACHE STRING "git ref of GB-Recomp/pokegold")
set(POKESILVER_REF   "main" CACHE STRING "git ref of GB-Recomp/pokesilver")
set(POKECRYSTAL_REF  "main" CACHE STRING "git ref of GB-Recomp/pokecrystal")

FetchContent_Declare(pokered     GIT_REPOSITORY https://github.com/GB-Recomp/pokered.git     GIT_TAG ${POKERED_REF}     GIT_SHALLOW TRUE)
FetchContent_Declare(pokeblue    GIT_REPOSITORY https://github.com/GB-Recomp/pokeblue.git    GIT_TAG ${POKEBLUE_REF}    GIT_SHALLOW TRUE)
FetchContent_Declare(pokeyellow  GIT_REPOSITORY https://github.com/GB-Recomp/pokeyellow.git  GIT_TAG ${POKEYELLOW_REF}  GIT_SHALLOW TRUE)
FetchContent_Declare(pokegold    GIT_REPOSITORY https://github.com/GB-Recomp/pokegold.git    GIT_TAG ${POKEGOLD_REF}    GIT_SHALLOW TRUE)
FetchContent_Declare(pokesilver  GIT_REPOSITORY https://github.com/GB-Recomp/pokesilver.git  GIT_TAG ${POKESILVER_REF}  GIT_SHALLOW TRUE)
FetchContent_Declare(pokecrystal GIT_REPOSITORY https://github.com/GB-Recomp/pokecrystal.git GIT_TAG ${POKECRYSTAL_REF} GIT_SHALLOW TRUE)

FetchContent_MakeAvailable(pokered pokeblue pokeyellow pokegold pokesilver pokecrystal)

# ---------------------------------------------------------------------------
# pokegb — the multi-game launcher executable
# ---------------------------------------------------------------------------
add_executable(pokegb launcher_main.cpp)
target_link_libraries(pokegb PRIVATE
    pokered_cart pokeblue_cart pokeyellow_cart
    pokegold_cart pokesilver_cart pokecrystal_cart
    gbrt
)

# IPO/LTO must stay OFF: gb-recompiled's runtime provides a strong
# gb_dispatch (in mock_ir.c) that intercepts each Gen 2 cart's
# ExchangeMysteryGiftData entry. With LTO inside gbrt, the linker can
# inline the dispatch into gb_step before honoring the strong definition.
