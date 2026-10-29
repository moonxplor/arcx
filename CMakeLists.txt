cmake_minimum_required(VERSION 3.16)
project(arcx VERSION 0.1.0 LANGUAGES C)

set(CMAKE_C_STANDARD 17)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

# Strict warnings
add_compile_options(
    -Wall -Wextra -Wpedantic
    -Wshadow -Wconversion
    -Wformat=2 -Wformat-security
    -Wnull-dereference
    -Wstrict-prototypes
)

# Sanitizer support for debug builds
option(ARCX_SANITIZE "Enable AddressSanitizer and UBSan" OFF)
if(ARCX_SANITIZE)
    add_compile_options(-fsanitize=address,undefined -fno-omit-frame-pointer)
    add_link_options(-fsanitize=address,undefined)
endif()

find_package(PkgConfig REQUIRED)
pkg_check_modules(NCURSES REQUIRED ncurses)
pkg_check_modules(SQLITE3 REQUIRED sqlite3)

set(ARCX_SOURCES
    src/main.c
    src/app.c
    src/ui.c
    src/system.c
    src/hyprland.c
    src/doctor.c
    src/quests.c
    src/focus.c
    src/network.c
    src/database.c
    src/config.c
)

add_executable(arcx ${ARCX_SOURCES})

target_include_directories(arcx PRIVATE
    ${CMAKE_SOURCE_DIR}/include
    ${NCURSES_INCLUDE_DIRS}
    ${SQLITE3_INCLUDE_DIRS}
)

target_link_libraries(arcx PRIVATE
    ${NCURSES_LIBRARIES}
    ${SQLITE3_LIBRARIES}
)

target_compile_definitions(arcx PRIVATE
    ARCX_VERSION="${PROJECT_VERSION}"
    ARCX_DATA_DIR="${CMAKE_SOURCE_DIR}/data"
    _POSIX_C_SOURCE=200809L
)

# Tests
enable_testing()

add_executable(test_system tests/test_system.c src/system.c)
target_include_directories(test_system PRIVATE ${CMAKE_SOURCE_DIR}/include)
target_compile_definitions(test_system PRIVATE _POSIX_C_SOURCE=200809L ARCX_VERSION="${PROJECT_VERSION}")

add_executable(test_quests tests/test_quests.c src/quests.c src/database.c)
target_include_directories(test_quests PRIVATE
    ${CMAKE_SOURCE_DIR}/include
    ${SQLITE3_INCLUDE_DIRS}
)
target_link_libraries(test_quests PRIVATE ${SQLITE3_LIBRARIES})
target_compile_definitions(test_quests PRIVATE
    _POSIX_C_SOURCE=200809L
    ARCX_VERSION="${PROJECT_VERSION}"
    ARCX_DATA_DIR="${CMAKE_SOURCE_DIR}/data"
)

add_executable(test_database tests/test_database.c src/database.c)
target_include_directories(test_database PRIVATE
    ${CMAKE_SOURCE_DIR}/include
    ${SQLITE3_INCLUDE_DIRS}
)
target_link_libraries(test_database PRIVATE ${SQLITE3_LIBRARIES})
target_compile_definitions(test_database PRIVATE _POSIX_C_SOURCE=200809L ARCX_VERSION="${PROJECT_VERSION}")

add_executable(test_config tests/test_config.c src/config.c)
target_include_directories(test_config PRIVATE ${CMAKE_SOURCE_DIR}/include)
target_compile_definitions(test_config PRIVATE _POSIX_C_SOURCE=200809L ARCX_VERSION="${PROJECT_VERSION}")

add_executable(test_cli tests/test_cli.c src/app.c src/config.c)
target_include_directories(test_cli PRIVATE ${CMAKE_SOURCE_DIR}/include)
target_compile_definitions(test_cli PRIVATE _POSIX_C_SOURCE=200809L ARCX_VERSION="${PROJECT_VERSION}")

add_test(NAME test_system COMMAND test_system)
add_test(NAME test_quests COMMAND test_quests)
add_test(NAME test_database COMMAND test_database)
add_test(NAME test_config COMMAND test_config)
add_test(NAME test_cli COMMAND test_cli)
