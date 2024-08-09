cmake_minimum_required(VERSION 3.14)

project(CustomQGCPlugin)

message(STATUS "Adding Custom Plugin")
find_package(Qt6 REQUIRED COMPONENTS Core)

# Enable custom build
set_property(DIRECTORY ${CMAKE_SOURCE_DIR}
    APPEND PROPERTY COMPILE_DEFINITIONS
    QGC_CUSTOM_BUILD
    CUSTOMHEADER="CustomPlugin.h"
    CUSTOMCLASS=CustomPlugin
)

# Ensure Python is found
find_package(Python3 COMPONENTS Interpreter)
if (Python3_FOUND)
    set(PYTHON_EXECUTABLE ${Python3_EXECUTABLE})
    message(STATUS "Python found: ${Python3_EXECUTABLE}")
else()
    find_package(PythonInterp)
    if (PYTHONINTERP_FOUND)
        set(PYTHON_EXECUTABLE ${PYTHON_EXECUTABLE})
        message(STATUS "Python found: ${PYTHON_EXECUTABLE}")
    else()
        message(FATAL_ERROR "Python not found")
    endif()
endif()

# Run the updateqrc.py script
if(PYTHON_EXECUTABLE)
    execute_process(
        COMMAND ${PYTHON_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/updateqrc.py
        RESULT_VARIABLE PYTHON_RESULT
        OUTPUT_VARIABLE PYTHON_OUTPUT
        ERROR_VARIABLE PYTHON_ERROR
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        OUTPUT_STRIP_TRAILING_WHITESPACE
    )
    message(STATUS "Python result: ${PYTHON_RESULT}")
    message(STATUS "Python output: ${PYTHON_OUTPUT}")
    message(STATUS "Python error: ${PYTHON_ERROR}")
endif()

# Our own, custom resources
list(APPEND CUSTOM_RESOURCES
    ${CMAKE_CURRENT_SOURCE_DIR}/qgcimages.qrc
    ${CMAKE_CURRENT_SOURCE_DIR}/qgcresources.qrc
    ${CMAKE_CURRENT_SOURCE_DIR}/qgroundcontrol.qrc
    ${CMAKE_CURRENT_SOURCE_DIR}/custom.qrc
)
set(QGC_RESOURCES ${CUSTOM_RESOURCES} PARENT_SCOPE)

set(QML_IMPORT_PATH "${CMAKE_CURRENT_SOURCE_DIR}/res" CACHE STRING "Extra qml import paths" PARENT_SCOPE)

qt_add_library(CustomPlugin STATIC
    src/CustomPlugin.cc
    src/CustomPlugin.h
    src/AutoPilotPlugin/CustomAutoPilotPlugin.cc
    src/AutoPilotPlugin/CustomAutoPilotPlugin.h
    src/FirmwarePlugin/CustomFirmwarePlugin.cc
    src/FirmwarePlugin/CustomFirmwarePlugin.h
    src/FirmwarePlugin/CustomFirmwarePluginFactory.cc
    src/FirmwarePlugin/CustomFirmwarePluginFactory.h
)

target_link_libraries(CustomPlugin
    PRIVATE
        FactSystem
        FirmwarePlugin
        PX4AutoPilotPlugin
        PX4FirmwarePlugin
        QmlControls
        Vehicle
    PUBLIC
        Qt6::Core
        MAVLink
        QGC
        Settings
)

target_include_directories(CustomPlugin
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/src
        ${CMAKE_CURRENT_SOURCE_DIR}/src/AutoPilotPlugin
        ${CMAKE_CURRENT_SOURCE_DIR}/src/FirmwarePlugin
        # Add include directories for QGroundControl source
        ${CMAKE_SOURCE_DIR}/libs
        ${CMAKE_SOURCE_DIR}/src
        ${CMAKE_SOURCE_DIR}/src/AutoPilotPlugins/PX4
        ${CMAKE_SOURCE_DIR}/src/FirmwarePlugin
        ${CMAKE_SOURCE_DIR}/src/Vehicle
        ${CMAKE_SOURCE_DIR}/src/FactSystem
        ${CMAKE_SOURCE_DIR}/src/Settings
        ${CMAKE_SOURCE_DIR}/src/QtLocationPlugin
        ${CMAKE_SOURCE_DIR}/src/QmlControls
        ${CMAKE_SOURCE_DIR}/src/MissionManager
)

