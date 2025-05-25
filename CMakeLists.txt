cmake_minimum_required(VERSION 3.22)
project(py-bitcoinkernel
    VERSION 0.1
    LANGUAGES CXX
)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED True)

include(GNUInstallDirs)

##############################################
# Configure and build bitcoinkernel dependency
##############################################
if(DEFINED ENV{BITCOINKERNEL_LIB})
    # This case implies that libbitcoinkernel and its .pc file are already installed
    # in a location findable by PkgConfig (e.g., system paths or PKG_CONFIG_PATH is set).
    # We'll proceed to the PkgConfig step directly.
    message(STATUS "Attempting to use pre-existing bitcoinkernel from environment (via PkgConfig)")

    find_package(PkgConfig REQUIRED)
    pkg_check_modules(bitcoinkernel REQUIRED IMPORTED_TARGET libbitcoinkernel)
    add_library(BitcoinCore::bitcoinkernel ALIAS PkgConfig::bitcoinkernel)
else()
    set(BITCOIN_SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/depend/bitcoin)
    message(STATUS "Building bitcoinkernel from source in ${BITCOIN_SOURCE_DIR}")

    if(NOT EXISTS ${BITCOIN_SOURCE_DIR}/CMakeLists.txt)
        message(FATAL_ERROR "Bitcoin source not found in ${BITCOIN_SOURCE_DIR}. Please ensure the submodule is initialized.")
    endif()

    set(ENABLE_WALLET OFF)
    set(BUILD_SHARED_LIBS OFF)
    set(BUILD_KERNEL_LIB ON)
    set(BUILD_DAEMON OFF)
    set(BUILD_CLI OFF)
    set(BUILD_TX OFF)
    set(BUILD_UTIL OFF)
    set(BUILD_TESTS OFF)
    add_subdirectory(${BITCOIN_SOURCE_DIR})
    add_library(BitcoinCore::bitcoinkernel ALIAS bitcoinkernel)  # TODO: This should be done in the upstream repo.
endif()

########################################################
# Configure and build nanobind and the pbk_ext extension
########################################################

find_package(Python 3.8 COMPONENTS Interpreter Development.Module REQUIRED)

execute_process(
    COMMAND "${Python_EXECUTABLE}" -m nanobind --cmake_dir
    OUTPUT_STRIP_TRAILING_WHITESPACE OUTPUT_VARIABLE nanobind_ROOT
)
find_package(nanobind CONFIG REQUIRED HINTS "${nanobind_ROOT}")


# TODO: The following code fails due to absence of `src/pbk_ext.cpp`.
return()


nanobind_add_module(
    pbk_ext
    STABLE_ABI
    src/pbk_ext.cpp
)

# TODO: At this point the `BitcoinCore::bitcoinkernel` target
#       is available. Use it directly.
target_link_libraries(pbk_ext PRIVATE BitcoinCore::bitcoinkernel)

install(TARGETS pbk_ext LIBRARY DESTINATION pbk)
