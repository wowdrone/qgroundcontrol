# Specify the minimum CMake version required
cmake_minimum_required(VERSION 3.22.1)

# Project name and version
project(QGroundControl VERSION 1.0 DESCRIPTION "QGroundControl Project" LANGUAGES C CXX)

message(STATUS "Adding Custom Plugin")
find_package(Qt6 REQUIRED COMPONENTS Core Quick)

# Enable custom build
set_property(DIRECTORY ${CMAKE_SOURCE_DIR}
    APPEND PROPERTY COMPILE_DEFINITIONS
    QGC_CUSTOM_BUILD
    CUSTOMHEADER="CustomPlugin.h"
    CUSTOMCLASS=CustomPlugin
)

# Find Python
find_package(Python3 COMPONENTS Interpreter)
if(Python3_FOUND)
    execute_process(
        COMMAND ${Python3_EXECUTABLE} ${CMAKE_CURRENT_SOURCE_DIR}/updateqrc.py
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        RESULT_VARIABLE PYTHON_RESULT
        OUTPUT_VARIABLE PYTHON_OUTPUT
        ERROR_VARIABLE PYTHON_ERROR
        OUTPUT_STRIP_TRAILING_WHITESPACE
    )
    message(STATUS "Python Result: ${PYTHON_RESULT}")
    message(STATUS "Python Output: ${PYTHON_OUTPUT}")
    message(STATUS "Python Error: ${PYTHON_ERROR}")
endif()

# Define custom resources
list(APPEND CUSTOM_RESOURCES
    ${CMAKE_CURRENT_SOURCE_DIR}/qgcimages.qrc
    ${CMAKE_CURRENT_SOURCE_DIR}/qgcresources.qrc
    ${CMAKE_CURRENT_SOURCE_DIR}/qgroundcontrol.qrc
    ${CMAKE_CURRENT_SOURCE_DIR}/custom.qrc
)
set(QGC_RESOURCES ${CUSTOM_RESOURCES} PARENT_SCOPE)

# Set QML import path
set(QML_IMPORT_PATH "${CMAKE_CURRENT_SOURCE_DIR}/res" CACHE STRING "Extra qml import paths" PARENT_SCOPE)

# Add custom plugin library
qt_add_library(CustomPlugin STATIC
    custom/src/CustomPlugin.cc
    custom/src/CustomPlugin.h
    custom/src/AutoPilotPlugin/CustomAutoPilotPlugin.cc
    custom/src/AutoPilotPlugin/CustomAutoPilotPlugin.h
    custom/src/FirmwarePlugin/CustomFirmwarePlugin.cc
    custom/src/FirmwarePlugin/CustomFirmwarePlugin.h
    custom/src/FirmwarePlugin/CustomFirmwarePluginFactory.cc
    custom/src/FirmwarePlugin/CustomFirmwarePluginFactory.h
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
        Qt6::Quick
        MAVLink
        QGC
        Settings
)

target_include_directories(CustomPlugin
    PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/custom/src
    ${CMAKE_CURRENT_SOURCE_DIR}/custom/src/AutoPilotPlugin
    ${CMAKE_CURRENT_SOURCE_DIR}/custom/src/FirmwarePlugin
    ${CMAKE_CURRENT_SOURCE_DIR}/src
    ${CMAKE_CURRENT_SOURCE_DIR}/src/AutoPilotPlugins
    ${CMAKE_CURRENT_SOURCE_DIR}/src/AutoPilotPlugins/PX4
    ${CMAKE_CURRENT_SOURCE_DIR}/src/FirmwarePlugin
    ${CMAKE_CURRENT_SOURCE_DIR}/src/Vehicle

# Add QML module if needed
file(GLOB_RECURSE QML_SOURCES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/res/*.qml)
file(GLOB QML_RESOURCES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/res/Images/*.*)

# AutoPilotPlugins 디렉토리의 소스 파일 추가
file(GLOB AUTOPILOT_SOURCES
    "${CMAKE_CURRENT_SOURCE_DIR}/src/AutoPilotPlugins/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/AutoPilotPlugins/*.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/AutoPilotPlugins/PX4/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/AutoPilotPlugins/PX4/*.h"
    )

# CustomPlugin 타겟에 AutoPilotPlugins 소스 추가
target_sources(CustomPlugin PRIVATE ${AUTOPILOT_SOURCES})
# CustomPlugin 타겟에 PX4 AutoPilotPlugin 소스 추가
target_sources(CustomPlugin PRIVATE ${PX4_AUTOPILOT_SOURCES})

# CustomPlugin에 대한 속성 설정
set_target_properties(CustomPlugin PROPERTIES 
    AUTOMOC ON
    AUTORCC ON
    AUTOUIC ON
)

# QML 모듈 추가
qt_add_qml_module(CustomPlugin
    URI QGroundControl.CustomPlugin
    VERSION 1.0
    RESOURCE_PREFIX "/"
    QML_FILES ${QML_SOURCES}
    RESOURCES ${QML_RESOURCES}
    OUTPUT_TARGETS CustomPlugin_targets
    IMPORT_PATH ${QML_IMPORT_PATH}
    IMPORTS
        QtQuick
        QtQuick.Controls
        QtQuick.Layouts
        QGroundControl
        QGroundControl.Controls
        QGroundControl.Palette
        QGroundControl.ScreenTools
    ENABLE_AUTOGEN_TOOLS ON
)

# AUTOMOC 설정 확인
get_target_property(_automoc CustomPlugin AUTOMOC)
message(STATUS "AUTOMOC for CustomPlugin: ${_automoc}")

# 모든 소스 파일에 대해 AUTOMOC 강제 적용
file(GLOB_RECURSE ALL_SOURCES 
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/*.h"
)
set_source_files_properties(${ALL_SOURCES} PROPERTIES SKIP_AUTOMOC OFF)


# Ensure QML import paths are set
set_target_properties(CustomPlugin PROPERTIES
    QML_IMPORT_PATHS "${QML_IMPORT_PATH}"
)

# Print QML import paths to debug
message(STATUS "QML Import Path: ${QML_IMPORT_PATH}")
