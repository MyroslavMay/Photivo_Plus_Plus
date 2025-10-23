#[[
This file is part of Photivo.

Photivo is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License version 3
as published by the Free Software Foundation.

Photivo is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with Photivo.  If not, see <http://www.gnu.org/licenses/>.
]]
cmake_minimum_required(VERSION 3.14 FATAL_ERROR)
include(FindPackageMessage)

option(WITH_SYSTEM_CIMG "use system provided CImg instead of the bundled one" OFF)
option(WITH_CLEAR "builds ptClear" ON)
option(WITH_ADOBE_PROFILES "builds Adobe profiles creator" OFF)
option(WITH_GIMP_TO_PT "installs GIMP-to-Photivo GIMP plugin" ON)
option(WITH_OLD_PT_TO_GIMP "builds old Photivo-to-GIMP plugin for GIMP 2.8 and older" OFF)
set(PT_QT_MAJOR_VERSION 6 CACHE STRING "major version of Qt to build against")

# Backwards compatibility with the WITH_GIMP option from the old CMake build system.
if (WITH_GIMP OR WITH_OLD_PT_TO_GIMP)
    set(WITH_OLD_GIMP ON)
else()
    set(WITH_OLD_GIMP OFF)
endif()

if (NOT "${PT_QT_MAJOR_VERSION}" MATCHES "^[56789]$")
    message(
        FATAL_ERROR
        "Unsupported Qt version '${PT_QT_MAJOR_VERSION}' requested. "
        "Must be a single digit greater or equal 5."
    )
endif()

find_package_message(
    photivo
"Using system CImg (WITH_SYSTEM_CIMG): ${WITH_SYSTEM_CIMG}
-- Building ptClear (WITH_CLEAR): ${WITH_CLEAR}
-- Building Adobe Profiles Creator (WITH_ADOBE_PROFILES): ${WITH_ADOBE_PROFILES}
-- Installing GIMP-to-Photivo plugin (WITH_GIMP_TO_PT): ${WITH_GIMP_TO_PT}
-- Building old Photivo-to-GIMP plugin (WITH_OLD_PT_TO_GIMP): ${WITH_OLD_GIMP}
-- Using Qt major version (PT_QT_MAJOR_VERSION): ${PT_QT_MAJOR_VERSION}"
    "${WITH_SYSTEM_CIMG}.${WITH_CLEAR}.${WITH_ADOBE_PROFILES}.${WITH_GIMP_TO_PT}.${WITH_OLD_GIMP}.${PT_QT_MAJOR_VERSION}"
)


project(photivo LANGUAGES C CXX)


# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# determine app version and Git commit hash

# If no .git folder exists, we can assume we are building from a source archive previously created
# with 'ninja package_source'.
if (EXISTS "${PROJECT_SOURCE_DIR}/.git")
    set(PT_HAVE_REPO TRUE)
    unset(PT_VERSION)

    find_package(Git)

    if (Git_FOUND)
        execute_process(
            COMMAND
                "${GIT_EXECUTABLE}"
                -C "${PROJECT_SOURCE_DIR}"
                log -n 1 "--pretty=format:%ad;%h" --date=short --color=never
            OUTPUT_STRIP_TRAILING_WHITESPACE
            OUTPUT_VARIABLE
                _commit_info
            RESULT_VARIABLE
                _exit_code
        )
        if ("${_exit_code}" EQUAL 0)
            # Git output formatted as a CMake list containing first the date, then the commit hash.
            list(GET _commit_info 1 PT_GIT_HASH)
            list(GET _commit_info 0 PT_VERSION)
            set(PT_VERSION "${PT_VERSION} (commit ${PT_GIT_HASH})")
        endif()
    endif()

    if (NOT DEFINED PT_VERSION)
        message(
            WARNING
            "Building from the Git repo, but either could not find Git, or Git exited with an error. "
            "Cannot determine Photivo's version info. Using current date/time instead."
        )
        string(TIMESTAMP PT_VERSION "compiled on %Y-%m-%d %H:%M:%S")
        set(PT_GIT_HASH "NOTFOUND")
    endif()
else()
    set(PT_HAVE_REPO FALSE)
    message(
        STATUS
        "Not building from the Git repo. Determining Photivo version from the directory name."
    )
    string(REGEX REPLACE ".*photivo-" "commit " PT_VERSION "${PROJECT_SOURCE_DIR}")
    set(PT_GIT_HASH "NOTFOUND")
endif()

if (NOT pt_version_reported)
    message(STATUS "Photivo version: ${PT_VERSION}")
    set(pt_version_reported TRUE CACHE INTERNAL "")
endif()


# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# dependencies

# --- build system dependencies
find_package(PkgConfig REQUIRED)

# --- compile/runtime dependencies
if (NOT OpenMP_FOUND)
    # Sadly OpenMP doesn’t do the already-found check internally.
    find_package(OpenMP REQUIRED)
endif()

set(min_qt_version)
if (${PT_QT_MAJOR_VERSION} EQUAL 5)
    set(min_qt_version 5.15)
endif()
find_package(Qt${PT_QT_MAJOR_VERSION} ${min_qt_version} REQUIRED COMPONENTS Widgets Network)
find_package_message(
    PT_QT
    "Found Qt: ${Qt${PT_QT_MAJOR_VERSION}_DIR} (found version \"${Qt${PT_QT_MAJOR_VERSION}Core_VERSION}\")"
    "${Qt${PT_QT_MAJOR_VERSION}_DIR}.${Qt${PT_QT_MAJOR_VERSION}Core_VERSION}"
)

find_package(JPEG REQUIRED)

pkg_check_modules(glib2   REQUIRED IMPORTED_TARGET glib-2.0>=2.18)
pkg_check_modules(lcms2   REQUIRED IMPORTED_TARGET lcms2)
pkg_check_modules(exiv2   REQUIRED IMPORTED_TARGET exiv2>=0.19)
pkg_check_modules(lqr     REQUIRED IMPORTED_TARGET lqr-1>=0.4.1)
pkg_check_modules(lensfun REQUIRED IMPORTED_TARGET lensfun>=0.2.5)
pkg_check_modules(GraphicsMagick     REQUIRED IMPORTED_TARGET GraphicsMagick++>=1.3.12)
pkg_check_modules(GraphicsMagickWand REQUIRED IMPORTED_TARGET GraphicsMagickWand>=1.3.12)
pkg_check_modules(fftw3   REQUIRED IMPORTED_TARGET fftw3>=3.2.2)
pkg_check_modules(libraw  REQUIRED IMPORTED_TARGET libraw>=0.19)

if (WITH_OLD_GIMP)
    pkg_check_modules(gimp REQUIRED IMPORTED_TARGET gimp-2.0>=2.6.10)
    pkg_check_modules(gtk2 REQUIRED IMPORTED_TARGET gtk+-2.0)
endif()


# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# Qt config

if (PT_QT_MAJOR_VERSION GREATER_EQUAL 6)
    qt_standard_project_setup()

    macro(pt_add_executable)
        qt_add_executable(${ARGV})
    endmacro()
else()
    # The qt_add_* commands are not available in Qt 5.
    macro(pt_add_executable)
        add_executable(${ARGV})
    endmacro()
endif()

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)


# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# global compiler config
# there may be more target specific config in the subdirs

if (MSVC)
    message(FATAL_ERROR "Photivo cannot be built with MSVC++. Use GCC instead.")
endif()

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED OFF)
set(CMAKE_CXX_EXTENSIONS ON)

add_compile_options(-Wno-deprecated-declarations)
include_directories("${CMAKE_CURRENT_LIST_DIR}/Sources")


# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# config install dirs

if (WIN32)
    set(CMAKE_INSTALL_BINDIR .)
    set(CMAKE_INSTALL_LIBDIR .)
    set(CMAKE_INSTALL_DATADIR .)
    set(PT_INSTALL_DATADIR "${CMAKE_INSTALL_DATADIR}")
else()
    include(GNUInstallDirs)
    set(PT_INSTALL_DATADIR "${CMAKE_INSTALL_DATADIR}/photivo")
endif()


# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# subdirectories

add_subdirectory(apps/photivo)

if (WITH_CLEAR)
    add_subdirectory(apps/ptClear)
endif()

if (WITH_ADOBE_PROFILES)
    add_subdirectory(apps/ptCreateAdobeProfiles)
endif()

if (WITH_OLD_GIMP)
    add_subdirectory(apps/photivo-to-gimp-plugin)
endif()


# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# installation

if (NOT WIN32)
    install(
        FILES
            "qrc/photivo-appicon.png"
        DESTINATION
            "${CMAKE_INSTALL_DATADIR}/pixmaps"
    )
    install(
        PROGRAMS
            "ReferenceMaterial/photivo.desktop"
        DESTINATION
            "${CMAKE_INSTALL_DATADIR}/applications"
    )
    if (WITH_GIMP_TO_PT)
        install(PROGRAMS "mm-extern-photivo.py" DESTINATION "${CMAKE_INSTALL_LIBDIR}/gimp/2.0/plug-ins")
    endif()
endif()

install(
    DIRECTORY
        "ChannelMixers"
        "Curves"
        "LensfunDatabase"
        "Presets"
        "Profiles"
        "Themes"
        "UISettings"
    DESTINATION
        "${PT_INSTALL_DATADIR}"
)
install(
    DIRECTORY
        "Translations"
    DESTINATION
        "${PT_INSTALL_DATADIR}"
    FILES_MATCHING PATTERN
        "*.qm"
)

if (NOT TARGET uninstall)
    configure_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
        "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
        IMMEDIATE @ONLY
    )

    add_custom_target(uninstall
        COMMAND "${CMAKE_COMMAND}" -P "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    )
endif()


# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# packaging

if (PT_HAVE_REPO)
    set(CPACK_SOURCE_IGNORE_FILES
        "_build.*/;build.*/;BUILD.*/;\\\\.git/;\\\\.gitignore$;\\\\.gitmodules$;\\\\.gitattributes")
    set(CPACK_PACKAGE_FILE_NAME "photivo-${PT_GIT_HASH}_${CMAKE_SYSTEM_PROCESSOR}")
    set(CPACK_SOURCE_PACKAGE_FILE_NAME "photivo-${PT_GIT_HASH}")
    set(CPACK_GENERATOR TBZ2)
    set(CPACK_SOURCE_GENERATOR TBZ2)
    include( CPack )
endif()
