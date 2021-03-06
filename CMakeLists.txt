set( QT_INSTALL_PATH ${CMAKE_PREFIX_PATH} )

cmake_minimum_required( VERSION 3.17 )
project( gtest_demo )

if( EXISTS ${CMAKE_BINARY_DIR}/conan_paths.cmake )
    include( ${CMAKE_BINARY_DIR}/conan_paths.cmake )
else()
    message( WARNING "The file conan_paths.cmake doesn't exist, you have to run conan install first" )
endif()

set( CMAKE_AUTOMOC ON )
set( CMAKE_AUTORCC ON )
set( CMAKE_AUTOUIC ON )

set( QT_VERSION 5 )
set( REQUIRED_LIBS Core )
set( REQUIRED_LIBS_QUALIFIED Qt5::Core )

add_executable( ${PROJECT_NAME} main.cpp )

if( NOT CMAKE_PREFIX_PATH )
    message( WARNING "CMAKE_PREFIX_PATH is not defined, you may need to set it "
             "(-DCMAKE_PREFIX_PATH=\"path/to/Qt/lib/cmake\" or -DCMAKE_PREFIX_PATH=/usr/include/{host}/qt{version}/ on Ubuntu)" )
endif()

find_package( Qt${QT_VERSION} COMPONENTS ${REQUIRED_LIBS} REQUIRED )
find_package( GTest REQUIRED )
target_link_libraries( ${PROJECT_NAME} PRIVATE ${REQUIRED_LIBS_QUALIFIED} GTest::gtest )
target_compile_features( ${PROJECT_NAME} PUBLIC cxx_std_14 )
if( WIN32 )
    set( DEBUG_SUFFIX )
    if( CMAKE_BUILD_TYPE MATCHES "Debug" )
        set( DEBUG_SUFFIX "d" )
    endif()
    if( NOT EXISTS "${QT_INSTALL_PATH}/bin" )
        set( QT_INSTALL_PATH "${QT_INSTALL_PATH}/.." )
        if( NOT EXISTS "${QT_INSTALL_PATH}/bin" )
            set( QT_INSTALL_PATH "${QT_INSTALL_PATH}/.." )
        endif()
    endif()
    if( EXISTS "${QT_INSTALL_PATH}/plugins/platforms/qwindows${DEBUG_SUFFIX}.dll" )
        add_custom_command( TARGET ${PROJECT_NAME} POST_BUILD
                            COMMAND ${CMAKE_COMMAND} -E make_directory
                            "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/platforms/" )
        add_custom_command( TARGET ${PROJECT_NAME} POST_BUILD
                            COMMAND ${CMAKE_COMMAND} -E copy
                            "${QT_INSTALL_PATH}/plugins/platforms/qwindows${DEBUG_SUFFIX}.dll"
                            "$<TARGET_FILE_DIR:${PROJECT_NAME}>/plugins/platforms/" )
    endif()
    foreach( QT_LIB ${REQUIRED_LIBS} )
        add_custom_command( TARGET ${PROJECT_NAME} POST_BUILD
                            COMMAND ${CMAKE_COMMAND} -E copy
                            "${QT_INSTALL_PATH}/bin/Qt${QT_VERSION}${QT_LIB}${DEBUG_SUFFIX}.dll"
                            "$<TARGET_FILE_DIR:${PROJECT_NAME}>" )
    endforeach( QT_LIB )
endif()
