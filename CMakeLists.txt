cmake_minimum_required(VERSION 3.5)

project(opengl VERSION 0.1 LANGUAGES CXX)

set(CMAKE_INCLUDE_CURRENT_DIR ON)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Widgets)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Widgets)

set(QRC_SOURCE_FILES ${CMAKE_CURRENT_LIST_DIR}/wall.jpeg)
qt5_add_resources(${QRC_SOURCE_FILES})
message("-----" ${QRC_SOURCE_FILES})

set(PROJECT_SOURCES
    main.cpp
    mainwindow.cpp
    mainwindow.h
    canvas.cpp
    canvas.h
    ${QRC_SOURCE_FILES}
)
source_group(TREE ${CMAKE_CURRENT_LIST_DIR} FILES ${PROJECT_SOURCES})

if(${QT_VERSION_MAJOR} GREATER_EQUAL 6)
    qt_add_executable(opengl
        MANUAL_FINALIZATION
        ${PROJECT_SOURCES}
    )

# Define target properties for Android with Qt 6 as:
# set_property(TARGET opengl APPEND PROPERTY QT_ANDROID_PACKAGE_SOURCE_DIR
# ${CMAKE_CURRENT_SOURCE_DIR}/android)
# For more information, see https://doc.qt.io/qt-6/qt-add-executable.html#target-creation
else()
    if(ANDROID)
        add_library(opengl SHARED
            ${PROJECT_SOURCES}
        )

    # Define properties for Android with Qt 5 after find_package() calls as:
    # set(ANDROID_PACKAGE_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/android")
    else()
        add_executable(opengl
            ${PROJECT_SOURCES}
        )
    endif()
endif()

target_link_libraries(opengl PRIVATE Qt${QT_VERSION_MAJOR}::Widgets)

set_target_properties(opengl PROPERTIES
    MACOSX_BUNDLE_GUI_IDENTIFIER my.example.com
    MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION}
    MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
    MACOSX_BUNDLE TRUE
    WIN32_EXECUTABLE TRUE
)

install(TARGETS opengl
    BUNDLE DESTINATION .
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})

if(QT_VERSION_MAJOR EQUAL 6)
    qt_finalize_executable(opengl)
endif()

# 获取qmake的指定属性
get_target_property(_qmake_executable Qt${QT_VERSION_MAJOR}::qmake IMPORTED_LOCATION)

# 反向找到bin目录
get_filename_component(_qt_bin_dir ${_qmake_executable} DIRECTORY)

# 从指定目录查找指定文件的路径
find_program(DEPLOYQT_EXECUTABLE NAMES windeployqt macdeployqt HINTS ${_qt_bin_dir})

# 增加部署命令
if(${CMAKE_GENERATOR} MATCHES "Visual Studio*")
    # vs中的EXECUTABLE_OUTPUT_PATH会自动追加build_type，使用vs的宏 $(var) 规避这个问题
    set(APP_REAL_PATH $(outdir))
else()
    set(APP_REAL_PATH ${EXECUTABLE_OUTPUT_PATH})
endif()

add_custom_command(TARGET opengl POST_BUILD
    COMMAND ${DEPLOYQT_EXECUTABLE} ${APP_REAL_PATH}
)