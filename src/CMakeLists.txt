set(CMAKE_CXX_STANDARD_REQUIRED True)

# If C++14 is available, use it as libtorent ABI depends on 11/14 version
if (cxx_std_14 IN_LIST CMAKE_CXX_COMPILE_FEATURES)
    message(STATUS "Building in C++14 mode")
    set(CMAKE_CXX_STANDARD "14")
else()
    message(STATUS "Building in C++11 mode")
    set(CMAKE_CXX_STANDARD "11")
endif()

include(MacroQbtCompilerSettings)
qbt_set_compiler_options()

include(QbtTargetSources)

find_package(Boost ${requiredBoostVersion} REQUIRED)
find_package(LibtorrentRasterbar ${requiredLibtorrentVersion} REQUIRED)
find_package(OpenSSL ${requiredOpensslVersion} REQUIRED)

if (Boost_VERSION VERSION_LESS 106000)
    add_definitions(-DBOOST_NO_CXX11_RVALUE_REFERENCES)
endif()

find_package(Qt5 ${requiredQtVersion} REQUIRED COMPONENTS Core Network Xml LinguistTools)
find_package(Qt5Widgets ${requiredQtVersion})
if (Qt5Widgets_FOUND)
    find_package(Qt5DBus ${requiredQtVersion})
else()
    add_definitions(-DDISABLE_GUI)
endif()

set_package_properties(Qt5Widgets PROPERTIES
    DESCRIPTION "Set of components for creating classic desktop-style UIs for the Qt5 framework"
    PURPOSE "Enables qBittorrent GUI. Unneeded for headless configuration."
    TYPE OPTIONAL
)

set_package_properties(Qt5DBus PROPERTIES
    DESCRIPTION "Qt5 module for inter-process communication over the D-Bus protocol"
    PURPOSE "Enables communication with other system components (e.g. notification service) via D-Bus. "
    TYPE RECOMMENDED
)

set(CMAKE_AUTOMOC True)
list(APPEND CMAKE_AUTORCC_OPTIONS -compress 9 -threshold 5)
if (APPLE)
    # Workaround CMake bug (autogen does not pass required parameters to moc)
    # Relevant issue: https://gitlab.kitware.com/cmake/cmake/issues/18041
    list(APPEND CMAKE_AUTOMOC_MOC_OPTIONS -DQ_OS_MAC -DQ_OS_DARWIN)
endif ()

include_directories(${CMAKE_CURRENT_SOURCE_DIR})

# defines
add_definitions(-DQT_NO_CAST_TO_ASCII)
# Efficient construction for QString & QByteArray (Qt >= 4.8)
add_definitions(-DQT_USE_QSTRINGBUILDER)

if (CMAKE_BUILD_TYPE MATCHES "Debug")
    message(STATUS "Project is built in DEBUG mode.")
else()
    message(STATUS "Project is built in RELEASE mode.")
    message(STATUS "Disabling debug output.")
    add_definitions(-DQT_NO_DEBUG_OUTPUT)
endif()

configure_file(config.h.cmakein ${CMAKE_CURRENT_BINARY_DIR}/config.h)

find_package(QtSingleApplication)
set_package_properties(QtSingleApplication PROPERTIES
    URL "https://code.qt.io/cgit/qt-solutions/qt-solutions.git/"
    DESCRIPTION "Qt library to start applications only once per user"
    TYPE RECOMMENDED
    PURPOSE "Use the system qtsingleapplication library or shipped one otherwise"
)

if (NOT QtSingleApplication_FOUND)
    add_subdirectory(app/qtsingleapplication)
endif ()

add_subdirectory(app)
add_subdirectory(base)

if (Qt5Widgets_FOUND)
    add_subdirectory(gui)
endif ()

if (WEBUI)
    add_subdirectory(webui)
endif (WEBUI)

