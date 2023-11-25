cmake_minimum_required(VERSION 3.22)


set(ProjectName ENTER_YOUR_PROJECT_NAME)    # 将里面的大写替换掉，输入你的项目名 
set(ComponentsList Core Gui Widgets Sql)    # 在这里输入你需要的库的名称


project(${ProjectName})
aux_source_directory(src SrcFiles)                  # 会自动找到其所在目录的src子文件夹，并包含内部的所有源码
file(GLOB_RECURSE SrcFiles "src/*.cpp" "src/*.h")

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

# 设置你的Qt安装目录 例如:
set(CMAKE_PREFIX_PATH "D:\\IDEs\\Qt\\6.6.1\\mingw_64")

find_package(Qt6 COMPONENTS
         ${ComponentsList}
        REQUIRED)

add_executable(${ProjectName}
        ${SrcFiles}
)

set(QtLibsWithNamespace)
foreach (component IN LISTS ComponentsList)
    list(APPEND QtLibsWithNamespace Qt::${component})
endforeach ()

target_link_libraries(${ProjectName}
        ${QtLibsWithNamespace}
)

if (WIN32 AND NOT DEFINED CMAKE_TOOLCHAIN_FILE)
    set(DEBUG_SUFFIX)
    if (MSVC AND CMAKE_BUILD_TYPE MATCHES "Debug")
        set(DEBUG_SUFFIX "d")
    endif ()
    set(QT_INSTALL_PATH "${CMAKE_PREFIX_PATH}")
    if (NOT EXISTS "${QT_INSTALL_PATH}/bin")
        set(QT_INSTALL_PATH "${QT_INSTALL_PATH}/..")
        if (NOT EXISTS "${QT_INSTALL_PATH}/bin")
            set(QT_INSTALL_PATH "${QT_INSTALL_PATH}/..")
        endif ()
    endif ()
    if (EXISTS "${QT_INSTALL_PATH}/plugins/platforms/qwindows${DEBUG_SUFFIX}.dll")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E make_directory
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/platforms/")
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/plugins/platforms/qwindows${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/platforms/")
    endif ()
    foreach (QT_LIB IN LISTS ComponentsList)
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy
                "${QT_INSTALL_PATH}/bin/Qt6${QT_LIB}${DEBUG_SUFFIX}.dll"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>")
    endforeach (QT_LIB)    
    if ("Sql" IN_LIST ComponentsList)
        add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy_directory
                "${QT_INSTALL_PATH}/plugins/sqldrivers/"
                "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/sqldrivers")
    endif ()
endif ()

