cmake_minimum_required(VERSION 3.22.1)

message(STATUS "Adding Custom Plugin")
find_package(Qt6 REQUIRED COMPONENTS Core)

# Enable custom build
set_property(DIRECTORY ${CMAKE_SOURCE_DIR}
    APPEND PROPERTY COMPILE_DEFINITIONS
    QGC_CUSTOM_BUILD
    CUSTOMHEADER="CustomPlugin.h"
    CUSTOMCLASS=CustomPlugin
)

find_package(Python)
if(PYTHON_FOUND)
    execute_process(
        COMMAND ${PYTHON_EXECUTABLE} updateqrc.py
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        RESULT_VARIABLE PYTHON_RESULT
        OUTPUT_VARIABLE PYTHON_OUTPUT
        ERROR_VARIABLE PYTHON_ERROR
        OUTPUT_STRIP_TRAILING_WHITESPACE
    )
    if(PYTHON_RESULT EQUAL 0)
        message(STATUS "Python script executed successfully.")
        message(STATUS "Python script output: ${PYTHON_OUTPUT}")
    else()
        message(WARNING "Python script execution failed with error: ${PYTHON_ERROR}")
    endif()
else()
    message(WARNING "Python not found, skipping script execution.")
endif()

# Our own, custom resources
list(APPEND CUSTOM_RESOURCES
    ${CMAKE_CURRENT_SOURCE_DIR}/qgcimages.qrc
    ${CMAKE_CURRENT_SOURCE_DIR}/qgcresources.qrc
    ${CMAKE_CURRENT_SOURCE_DIR}/qgroundcontrol.qrc
    ${CMAKE_CURRENT_SOURCE_DIR}/custom.qrc
)
set(QGC_RESOURCES ${CUSTOM_RESOURCES})

# Set QML import path
set(QML_IMPORT_PATH "${CMAKE_CURRENT_SOURCE_DIR}/res" CACHE STRING "Extra qml import paths")

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
)

# qt_add_qml_module은 필요한 경우 주석을 해제하여 사용하십시오
# file(GLOB_RECURSE QML_SOURCES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/res/*.qml)
# file(GLOB QML_RESOURCES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/res/Images/*.*)
# qt_add_qml_module(CustomPlugin
#     URI QGroundControl.CustomPlugin
#     VERSION 1.0
#     RESOURCE_PREFIX "/"
#     QML_FILES ${QML_SOURCES}
#     RESOURCES ${QML_RESOURCES}
#     OUTPUT_TARGETS CustomPlugin_targets
#     IMPORT_PATH ${QT_QML_OUTPUT_DIRECTORY}
#     IMPORTS
#         QtQuick
#         QtQuick.Controls
#         QtQuick.Layouts
#         QGroundControl
#         QGroundControl.Controls
#         QGroundControl.Palette
#         QGroundControl.ScreenTools
# )
