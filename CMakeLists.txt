cmake_minimum_required(VERSION 3.15...3.18)

# -- Project -------------------------------------------------------------------

# Extract the version info from the header file.
file(STRINGS source/sqlite3.h _ver_line
     REGEX "^#define SQLITE_VERSION  *\"[0-9]+\\.[0-9]+\\.[0-9]+\""
     LIMIT_COUNT 1)
string(REGEX MATCH "[0-9]+\\.[0-9]+\\.[0-9]+"
       SQLITE_VERSION "${_ver_line}")
unset(_ver_line)

project(SQLite3
        VERSION ${SQLITE_VERSION}
        DESCRIPTION "SQLite: an embedded SQL database engine"
        HOMEPAGE_URL "https://github.com/algoritnl/sqlite-amalgamation"
        LANGUAGES C)

# -- Configuration -------------------------------------------------------------

option(BUILD_SHARED_LIBS "Build shared libraries." OFF)

set(SQLITE_THREADSAFE "1" CACHE STRING
    "Select the thread model: 0 = single-threaded; 1 = serialized (default); 2 = multi-threaded.")
set(ACCEPTABLE_THREADSAFE_VALUES "0;1;2")
set_property(CACHE SQLITE_THREADSAFE PROPERTY STRINGS ${ACCEPTABLE_THREADSAFE_VALUES})
if(NOT SQLITE_THREADSAFE IN_LIST ACCEPTABLE_THREADSAFE_VALUES)
    message(WARNING
        "SQLITE_THREADSAFE value \"${SQLITE_THREADSAFE}\" is not valid.\n"
        "Continuing with the default: 1 (serialized mode).")
    set_property(CACHE SQLITE_THREADSAFE PROPERTY VALUE "1")
    set(SQLITE_THREADMODEL_SUFFIX "")
elseif(SQLITE_THREADSAFE STREQUAL "0")
    message(STATUS "Building the SQLite3 library in single-threading mode")
    set(SQLITE_THREADMODEL_SUFFIX "-st")
elseif(SQLITE_THREADSAFE STREQUAL "1")
    message(STATUS "Building the SQLite3 library in serialized mode")
    set(SQLITE_THREADMODEL_SUFFIX "")
elseif(SQLITE_THREADSAFE STREQUAL "2")
    message(STATUS "Building the SQLite3 library in multi-threading mode")
    set(SQLITE_THREADMODEL_SUFFIX "-mt")
else()
    message(FATAL_ERROR "Missing logic. Please update.")
endif()

set(SQLITE_DEFAULT_MEMSTATUS "1" CACHE STRING
    "Enable the SQLITE_CONFIG_MEMSTATUS features by default.")
set(ACCEPTABLE_DEFAULT_MEMSTATUS_VALUES "0;1")
set_property(CACHE SQLITE_DEFAULT_MEMSTATUS PROPERTY STRINGS ${ACCEPTABLE_DEFAULT_MEMSTATUS_VALUES})
if(NOT SQLITE_DEFAULT_MEMSTATUS IN_LIST ACCEPTABLE_DEFAULT_MEMSTATUS_VALUES)
    message(WARNING
        "SQLITE_DEFAULT_MEMSTATUS value \"${SQLITE_DEFAULT_MEMSTATUS}\" is not valid.\n"
        "Continuing with the default: 1 (Enabled).")
    set_property(CACHE SQLITE_DEFAULT_MEMSTATUS PROPERTY VALUE "1")
endif()

set(SQLITE_DEFAULT_SYNCHRONOUS "2" CACHE STRING
    "Determine the default value of PRAGMA synchronous.")
set(ACCEPTABLE_SYNCHRONOUS_VALUES "0;1;2;3")
set_property(CACHE SQLITE_DEFAULT_SYNCHRONOUS PROPERTY STRINGS ${ACCEPTABLE_SYNCHRONOUS_VALUES})
if(NOT SQLITE_DEFAULT_SYNCHRONOUS IN_LIST ACCEPTABLE_SYNCHRONOUS_VALUES)
    message(WARNING
        "SQLITE_DEFAULT_SYNCHRONOUS value \"${SQLITE_DEFAULT_SYNCHRONOUS}\" is not valid.\n"
        "Continuing with the default: 2 (FULL).")
    set_property(CACHE SQLITE_DEFAULT_SYNCHRONOUS PROPERTY VALUE "2")
endif()

set(SQLITE_DEFAULT_WAL_SYNCHRONOUS "-1" CACHE STRING
    "Determine the default value of PRAGMA synchronous in WAL mode.")
set(ACCEPTABLE_WAL_SYNCHRONOUS_VALUES "-1;0;1;2;3")
set_property(CACHE SQLITE_DEFAULT_WAL_SYNCHRONOUS PROPERTY STRINGS ${ACCEPTABLE_WAL_SYNCHRONOUS_VALUES})
if(NOT SQLITE_DEFAULT_WAL_SYNCHRONOUS IN_LIST ACCEPTABLE_WAL_SYNCHRONOUS_VALUES)
    message(WARNING
        "SQLITE_DEFAULT_WAL_SYNCHRONOUS value \"${SQLITE_DEFAULT_WAL_SYNCHRONOUS}\" is not valid.\n"
        "Continuing with the default: -1 (follow SQLITE_DEFAULT_SYNCHRONOUS).")
    set_property(CACHE SQLITE_DEFAULT_WAL_SYNCHRONOUS PROPERTY VALUE "-1")
    set(SQLITE_DEFAULT_WAL_SYNCHRONOUS ${SQLITE_DEFAULT_SYNCHRONOUS})
elseif(SQLITE_DEFAULT_WAL_SYNCHRONOUS STREQUAL "-1")
    set(SQLITE_DEFAULT_WAL_SYNCHRONOUS ${SQLITE_DEFAULT_SYNCHRONOUS})
endif()

set(SQLITE_DQS "3" CACHE STRING
    "Enable DQS: 0 = None (recommended); 1 = DML; 2 = DDL; 3 = Both (default).")
set(ACCEPTABLE_DQS_VALUES "0;1;2;3")
set_property(CACHE SQLITE_DQS PROPERTY STRINGS ${ACCEPTABLE_DQS_VALUES})
if(NOT SQLITE_DQS IN_LIST ACCEPTABLE_DQS_VALUES)
    message(WARNING
        "SQLITE_DQS value \"${SQLITE_DQS}\" is not valid.\n"
        "Continuing with the default: 3 (Both).")
    set_property(CACHE SQLITE_DQS PROPERTY VALUE "3")
endif()

option(SQLITE_LIKE_DOESNT_MATCH_BLOBS "Operators LIKE and GLOB always return FALSE if either operand is a BLOB." OFF)

set(SQLITE_MAX_EXPR_DEPTH "-1" CACHE STRING
    "Set the maximum expression tree depth.")
if(NOT SQLITE_MAX_EXPR_DEPTH GREATER_EQUAL "-1")
    message(WARNING
        "SQLITE_MAX_EXPR_DEPTH value \"${SQLITE_MAX_EXPR_DEPTH}\" is not valid.\n"
        "Continuing with the default: -1 (use the implementation's own default).")
    set_property(CACHE SQLITE_MAX_EXPR_DEPTH PROPERTY VALUE "-1")
endif()

option(SQLITE_ENABLE_COLUMN_METADATA "Enable some extra APIs that are required by some common systems, including Ruby-on-Rails." OFF)
option(SQLITE_OMIT_DECLTYPE "Omit the ability to return the declared type of columns from queries." OFF)
if(SQLITE_ENABLE_COLUMN_METADATA AND SQLITE_OMIT_DECLTYPE)
    message(SEND_ERROR "Options SQLITE_ENABLE_COLUMN_META and SQLITE_OMIT_DECLTYPE are mutually exclusive.")
endif()

option(SQLITE_ENABLE_DBSTAT_VTAB "This option enables the dbstat virtual table." OFF)
option(SQLITE_ENABLE_EXPLAIN_COMMENTS "This option adds extra logic to SQLite that inserts comment text into the output of EXPLAIN." OFF)
option(SQLITE_ENABLE_FTS3 "Enable version 3 of the full-text search engine." OFF)
option(SQLITE_ENABLE_FTS3_PARENTHESIS "Enable operators AND and NOT and allow nested parentheses." OFF)
option(SQLITE_ENABLE_FTS4 "Enable versions 3 and 4 of the full-text search engine." OFF)
option(SQLITE_ENABLE_FTS5 "Enable version 5 of the full-text search engine." OFF)
option(SQLITE_ENABLE_GEOPOLY "Enable the Geopoly extension." OFF)
option(SQLITE_ENABLE_ICU "Enable the ICU (International Components for Unicode) extension" OFF)
option(SQLITE_ENABLE_JSON1 "Enable the JSON SQL functions." OFF)
option(SQLITE_ENABLE_RBU "Enable the code that implements the RBU extension." OFF)
option(SQLITE_ENABLE_RTREE "This option causes SQLite to include support for the R*Tree index extension." OFF)
option(SQLITE_ENABLE_STMTVTAB "Enable the SQLITE_STMT virtual table logic." OFF)

option(SQLITE_OMIT_DEPRECATED "Omit deprecated interfaces and features." OFF)
option(SQLITE_OMIT_LOAD_EXTENSION "Omit the extension loading mechanism." OFF)
option(SQLITE_OMIT_PROGRESS_CALLBACK "Omit the capability to issue progress callbacks." OFF)
option(SQLITE_OMIT_SHARED_CACHE "Omit support for shared-cache mode." OFF)
option(SQLITE_USE_ALLOCA "Use alloca for memory allocations if possible." OFF)

option(SQLITE_BUILD_SHELL "Build the interactive shell." ON)
option(SQLITE_HAVE_ZLIB "Enable compression and decompression functions in the command-line shell." OFF)

option(STRICT_WARNINGS "Enable strict warnings." OFF)
mark_as_advanced(STRICT_WARNINGS)

include(GNUInstallDirs)

# -- Library -------------------------------------------------------------------

add_library(SQLite3)
add_library(SQLite::SQLite3 ALIAS SQLite3)

set_target_properties(SQLite3 PROPERTIES
    OUTPUT_NAME sqlite3${SQLITE_THREADMODEL_SUFFIX}
    DEBUG_POSTFIX -dbg
    VERSION ${SQLite3_VERSION}
    SOVERSION ${SQLite3_VERSION}
    # This property is needed on Windows to create the import library.
    WINDOWS_EXPORT_ALL_SYMBOLS ON)

target_sources(SQLite3
    PRIVATE source/sqlite3.c)

target_compile_definitions(SQLite3
    PUBLIC
        $<$<BOOL:${SQLITE_OMIT_DEPRECATED}>:SQLITE_OMIT_DEPRECATED>
        $<$<BOOL:${SQLITE_OMIT_LOAD_EXTENSION}>:SQLITE_OMIT_LOAD_EXTENSION>
    PRIVATE
        SQLITE_DEFAULT_MEMSTATUS=${SQLITE_DEFAULT_MEMSTATUS}
        SQLITE_DEFAULT_SYNCHRONOUS=${SQLITE_DEFAULT_SYNCHRONOUS}
        SQLITE_DEFAULT_WAL_SYNCHRONOUS=${SQLITE_DEFAULT_WAL_SYNCHRONOUS}
        SQLITE_DQS=${SQLITE_DQS}
        $<$<NOT:$<STREQUAL:${SQLITE_MAX_EXPR_DEPTH},-1>>:SQLITE_MAX_EXPR_DEPTH=${SQLITE_MAX_EXPR_DEPTH}>
        SQLITE_THREADSAFE=${SQLITE_THREADSAFE}
        $<$<BOOL:${SQLITE_ENABLE_COLUMN_METADATA}>:SQLITE_ENABLE_COLUMN_METADATA>
        $<$<BOOL:${SQLITE_ENABLE_DBSTAT_VTAB}>:SQLITE_ENABLE_DBSTAT_VTAB>
        $<$<BOOL:${SQLITE_ENABLE_EXPLAIN_COMMENTS}>:SQLITE_ENABLE_EXPLAIN_COMMENTS>
        $<$<BOOL:${SQLITE_ENABLE_FTS3}>:SQLITE_ENABLE_FTS3>
        $<$<BOOL:${SQLITE_ENABLE_FTS3_PARENTHESIS}>:SQLITE_ENABLE_FTS3_PARENTHESIS>
        $<$<BOOL:${SQLITE_ENABLE_FTS4}>:SQLITE_ENABLE_FTS4>
        $<$<BOOL:${SQLITE_ENABLE_FTS5}>:SQLITE_ENABLE_FTS5>
        $<$<BOOL:${SQLITE_ENABLE_GEOPOLY}>:SQLITE_ENABLE_GEOPOLY>
        $<$<BOOL:${SQLITE_ENABLE_ICU}>:SQLITE_ENABLE_ICU>
        $<$<BOOL:${SQLITE_ENABLE_JSON1}>:SQLITE_ENABLE_JSON1>
        $<$<BOOL:${SQLITE_ENABLE_RBU}>:SQLITE_ENABLE_RBU>
        $<$<BOOL:${SQLITE_ENABLE_RTREE}>:SQLITE_ENABLE_RTREE>
        $<$<BOOL:${SQLITE_ENABLE_STMTVTAB}>:SQLITE_ENABLE_STMTVTAB>
        $<$<BOOL:${SQLITE_LIKE_DOESNT_MATCH_BLOBS}>:SQLITE_LIKE_DOESNT_MATCH_BLOBS>
        $<$<BOOL:${SQLITE_OMIT_DECLTYPE}>:SQLITE_OMIT_DECLTYPE>
        $<$<BOOL:${SQLITE_OMIT_PROGRESS_CALLBACK}>:SQLITE_OMIT_PROGRESS_CALLBACK>
        $<$<BOOL:${SQLITE_OMIT_SHARED_CACHE}>:SQLITE_OMIT_SHARED_CACHE>
        $<$<BOOL:${SQLITE_USE_ALLOCA}>:SQLITE_USE_ALLOCA>
        )

target_include_directories(SQLite3
    INTERFACE
        "$<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/source>"
        "$<INSTALL_INTERFACE:include>")

if(UNIX)
    find_library(MATH_LIBRARY m)
    if(MATH_LIBRARY)
        message(VERBOSE "Found Math: ${MATH_LIBRARY}")
        target_link_libraries(SQLite3
            INTERFACE $<$<NOT:$<BOOL:${BUILD_SHARED_LIBS}>>:${MATH_LIBRARY}>
            PRIVATE $<$<BOOL:${BUILD_SHARED_LIBS}>:${MATH_LIBRARY}>)
    endif()
endif()

if(NOT SQLITE_THREADSAFE STREQUAL "0")
    set(THREADS_PREFER_PTHREAD_FLAG TRUE)
    find_package(Threads REQUIRED)
    target_link_libraries(SQLite3
        INTERFACE $<$<NOT:$<BOOL:${BUILD_SHARED_LIBS}>>:Threads::Threads>
        PRIVATE $<$<BOOL:${BUILD_SHARED_LIBS}>:Threads::Threads>)
endif()

if(NOT SQLITE_OMIT_LOAD_EXTENSION)
    target_link_libraries(SQLite3
        INTERFACE $<$<NOT:$<BOOL:${BUILD_SHARED_LIBS}>>:${CMAKE_DL_LIBS}>
        PRIVATE $<$<BOOL:${BUILD_SHARED_LIBS}>:${CMAKE_DL_LIBS}>)
endif()

if(SQLITE_ENABLE_ICU)
    find_package(ICU REQUIRED COMPONENTS i18n uc)
    target_link_libraries(SQLite3
        INTERFACE "$<$<NOT:$<BOOL:${BUILD_SHARED_LIBS}>>:ICU::i18n;ICU::uc>"
        PRIVATE "$<$<BOOL:${BUILD_SHARED_LIBS}>:ICU::i18n;ICU::uc>")
endif()

include(CheckIncludeFiles)
include(CheckSymbolExists)
include(CheckTypeSize)

check_include_files(inttypes.h HAVE_INTTYPES_H)
check_include_files(stdint.h HAVE_STDINT_H)
check_type_size(int8_t INT8_T)
check_type_size(int16_t INT16_T)
#check_type_size(int32_t INT32_T)
check_type_size(uint8_t UINT8_T)
check_type_size(uint16_t UINT16_T)
check_type_size(uint32_t UINT32_T)
target_compile_definitions(SQLite3
    PRIVATE
        $<$<BOOL:${HAVE_INTTYPES_H}>:HAVE_INTTYPES_H>
        $<$<BOOL:${HAVE_STDINT_H}>:HAVE_STDINT_H>
        $<$<BOOL:${HAVE_INT8_T}>:HAVE_INT8_T>
        $<$<BOOL:${HAVE_INT16_T}>:HAVE_INT16_T>
        #$<$<BOOL:${HAVE_INT32_T}>:HAVE_INT32_T>
        $<$<BOOL:${HAVE_UINT8_T}>:HAVE_UINT8_T>
        $<$<BOOL:${HAVE_UINT16_T}>:HAVE_UINT16_T>
        $<$<BOOL:${HAVE_UINT32_T}>:HAVE_UINT32_T>)

# fcntl.h sys/stat.h
check_symbol_exists(posix_fallocate fcntl.h HAVE_POSIX_FALLOCATE)
check_symbol_exists(lstat "fcntl.h;sys/stat.h" HAVE_LSTAT)
target_compile_definitions(SQLite3
    PRIVATE
        $<$<BOOL:${HAVE_POSIX_FALLOCATE}>:HAVE_POSIX_FALLOCATE>
        $<$<BOOL:${HAVE_LSTAT}>:HAVE_LSTAT>)

check_include_files(malloc.h HAVE_MALLOC_H)
if(HAVE_MALLOC_H)
    check_symbol_exists(malloc_usable_size malloc.h HAVE_MALLOC_USABLE_SIZE)
    target_compile_definitions(SQLite3
        PRIVATE
            $<$<BOOL:${HAVE_MALLOC_H}>:HAVE_MALLOC_H>
            $<$<BOOL:${HAVE_MALLOC_USABLE_SIZE}>:HAVE_MALLOC_USABLE_SIZE>)
endif()

# math.h
check_symbol_exists(isnan math.h HAVE_ISNAN)
target_compile_definitions(SQLite3
    PRIVATE
        $<$<BOOL:${HAVE_ISNAN}>:HAVE_ISNAN>)

# string.h
check_symbol_exists(strchrnul string.h HAVE_ISNAN)
check_symbol_exists(strerror_r string.h HAVE_STRERROR_R)
target_compile_definitions(SQLite3
    PRIVATE
        $<$<BOOL:${HAVE_STRCHRNUL}>:HAVE_STRCHRNUL>
        $<$<BOOL:${HAVE_STRERROR_R}>:HAVE_STRERROR_R>)

# time.h
check_symbol_exists(gmtime_r time.h HAVE_GMTIME_R)
check_symbol_exists(localtime_r time.h HAVE_LOCALTIME_R)
check_symbol_exists(localtime_s time.h HAVE_LOCALTIME_S)
target_compile_definitions(SQLite3
    PRIVATE
        $<$<BOOL:${HAVE_GMTIME_R}>:HAVE_GMTIME_R>
        $<$<BOOL:${HAVE_LOCALTIME_R}>:HAVE_LOCALTIME_R>
        $<$<BOOL:${HAVE_LOCALTIME_S}>:HAVE_LOCALTIME_S>)

check_include_files(unistd.h HAVE_UNISTD_H)
if(HAVE_UNISTD_H)
    check_symbol_exists(fdatasync unistd.h HAVE_FDATASYNC)
    check_symbol_exists(gethostuuid unistd.h HAVE_GETHOSTUUID)
    check_symbol_exists(pread unistd.h HAVE_PREAD)
    check_symbol_exists(pread64 unistd.h HAVE_PREAD64)
    check_symbol_exists(pwrite unistd.h HAVE_PWRITE)
    check_symbol_exists(pwrite64 unistd.h HAVE_PWRITE64)
    check_symbol_exists(readlink unistd.h HAVE_READLINK)
    check_symbol_exists(usleep unistd.h HAVE_USLEEP)
    target_compile_definitions(SQLite3
        PRIVATE
            $<$<BOOL:${HAVE_FDATASYNC}>:HAVE_FDATASYNC>
            $<$<BOOL:${HAVE_GETHOSTUUID}>:HAVE_GETHOSTUUID>
            $<$<BOOL:${HAVE_PREAD}>:HAVE_PREAD>
            $<$<BOOL:${HAVE_PREAD64}>:HAVE_PREAD64>
            $<$<BOOL:${HAVE_PWRITE}>:HAVE_PWRITE>
            $<$<BOOL:${HAVE_PWRITE64}>:HAVE_PWRITE64>
            $<$<BOOL:${HAVE_READLINK}>:HAVE_READLINK>
            $<$<BOOL:${HAVE_USLEEP}>:HAVE_USLEEP>)
endif()

check_symbol_exists(utime utime.h HAVE_UTIME)
target_compile_definitions(SQLite3
    PRIVATE
        $<$<BOOL:${HAVE_UTIME}>:HAVE_UTIME>)

if(STRICT_WARNINGS)
    target_compile_options(SQLite3
        PRIVATE
            # Enable as many warnings as possible.
            $<$<C_COMPILER_ID:Clang,GNU>:-Wpedantic>
            $<$<C_COMPILER_ID:Clang,GNU>:-Wall>
            $<$<C_COMPILER_ID:Clang,GNU>:-Wextra>
            $<$<C_COMPILER_ID:MSVC>:/Wall>
            # Disable individual warnings found during compilation.
            $<$<C_COMPILER_ID:GNU>:-Wno-implicit-fallthrough>
            $<$<C_COMPILER_ID:Clang,GNU>:-Wno-sign-compare>
            $<$<C_COMPILER_ID:Clang,GNU>:-Wno-unused-parameter>
            $<$<C_COMPILER_ID:MSVC>:/wd4242>
            $<$<C_COMPILER_ID:MSVC>:/wd4255>
            $<$<C_COMPILER_ID:MSVC>:/wd4267>
            $<$<C_COMPILER_ID:MSVC>:/wd4574>
            $<$<C_COMPILER_ID:MSVC>:/wd4668>
            $<$<C_COMPILER_ID:MSVC>:/wd4711>
            $<$<C_COMPILER_ID:MSVC>:/wd4820>
            $<$<C_COMPILER_ID:MSVC>:/wd5045>)
endif()

install(TARGETS SQLite3
        EXPORT SQLite3Targets
        RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
        LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
        ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}")

install(FILES source/sqlite3.h
              source/sqlite3ext.h
        DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}")

# -- Interactive shell ---------------------------------------------------------

if(SQLITE_BUILD_SHELL)
    add_executable(SQLite3Shell)

    set_target_properties(SQLite3Shell PROPERTIES
        OUTPUT_NAME sqlite3
        INSTALL_RPATH "$ORIGIN/../lib")

    target_sources(SQLite3Shell
        PRIVATE source/shell.c)

    target_link_libraries(SQLite3Shell
        PRIVATE SQLite::SQLite3)

    if(UNIX)
        find_package(PkgConfig)
        if(PKG_CONFIG_FOUND)
            pkg_search_module(Readline IMPORTED_TARGET readline)
            if(Readline_FOUND)
                target_compile_definitions(SQLite3Shell PRIVATE HAVE_READLINE)
                target_link_libraries(SQLite3Shell PRIVATE PkgConfig::Readline)
            endif()
        endif()
    endif()

    if(SQLITE_HAVE_ZLIB)
        find_package(ZLIB REQUIRED)
        target_compile_definitions(SQLite3Shell PRIVATE SQLITE_HAVE_ZLIB)
        target_link_libraries(SQLite3Shell PRIVATE ZLIB::ZLIB)
    endif()

    if(STRICT_WARNINGS)
        target_compile_options(SQLite3Shell
            PRIVATE
                # Enable as many warnings as possible.
                $<$<C_COMPILER_ID:Clang,GNU>:-Wpedantic>
                $<$<C_COMPILER_ID:Clang,GNU>:-Wall>
                $<$<C_COMPILER_ID:Clang,GNU>:-Wextra>
                $<$<C_COMPILER_ID:MSVC>:/Wall>
                # Disable individual warnings found during compilation.
                $<$<C_COMPILER_ID:GNU>:-Wno-implicit-fallthrough>
                $<$<C_COMPILER_ID:Clang,GNU>:-Wno-missing-field-initializers>
                $<$<C_COMPILER_ID:GNU>:-Wno-stringop-overflow>
                $<$<C_COMPILER_ID:Clang,GNU>:-Wno-unused-parameter>
                $<$<C_COMPILER_ID:MSVC>:/wd4242>
                $<$<C_COMPILER_ID:MSVC>:/wd4255>
                $<$<C_COMPILER_ID:MSVC>:/wd4668>
                $<$<C_COMPILER_ID:MSVC>:/wd4710>
                $<$<C_COMPILER_ID:MSVC>:/wd4711>
                $<$<C_COMPILER_ID:MSVC>:/wd4774>
                $<$<C_COMPILER_ID:MSVC>:/wd4820>
                $<$<C_COMPILER_ID:MSVC>:/wd5045>)
    else()
        target_compile_options(SQLite3Shell
            PRIVATE
                # Disable warnings that are present without strict warnings enabled.
                $<$<C_COMPILER_ID:GNU>:-Wno-stringop-overflow>)
    endif()

    install(TARGETS SQLite3Shell
            RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}")
endif()

# -- Testing -------------------------------------------------------------------

include(CTest)

if(BUILD_TESTING AND SQLITE_BUILD_SHELL)
    add_test(NAME CheckVersion
        COMMAND sqlite3 -version)
    set_tests_properties(CheckVersion PROPERTIES
        PASS_REGULAR_EXPRESSION ${SQLite3_VERSION})
endif()

# -- Export --------------------------------------------------------------------

install(EXPORT SQLite3Targets
        FILE SQLite3Targets.cmake
        DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/SQLite3"
        NAMESPACE SQLite::)

export(EXPORT SQLite3Targets
       FILE "${CMAKE_CURRENT_BINARY_DIR}/SQLite3Targets.cmake")

include(CMakePackageConfigHelpers)

configure_package_config_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake/SQLite3Config.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/SQLite3Config.cmake"
    INSTALL_DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/SQLite3"
    NO_SET_AND_CHECK_MACRO
    NO_CHECK_REQUIRED_COMPONENTS_MACRO)

write_basic_package_version_file(
    "${CMAKE_CURRENT_BINARY_DIR}/SQLite3ConfigVersion.cmake"
    COMPATIBILITY AnyNewerVersion)

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/SQLite3Config.cmake"
              "${CMAKE_CURRENT_BINARY_DIR}/SQLite3ConfigVersion.cmake"
        DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/SQLite3")

# -- Packaging -----------------------------------------------------------------

include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/UNLICENSE.md")
set(CPACK_RESOURCE_FILE_README "${CMAKE_CURRENT_SOURCE_DIR}/README.md")
set(CPACK_PACKAGE_VERSION_MAJOR "${SQLite3_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${SQLite3_VERSION_MINOR}")
set(CPACK_PACKAGE_VERSION_PATCH "${SQLite3_VERSION_PATCH}")
include(CPack)
