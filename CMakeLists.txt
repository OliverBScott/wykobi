cmake_minimum_required(VERSION 3.8)

project(wykobi VERSION 0.0.5
    HOMEPAGE_URL "https://www.wykobi.com/index.html"
    LANGUAGES CXX)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

if(NOT WYKOBI_FIND_QUIETLY)
    message(STATUS "wykobi v${wykobi_VERSION}")
endif()

if(CMAKE_SOURCE_DIR STREQUAL PROJECT_SOURCE_DIR AND NOT DEFINED WYKOBI_MASTER_PROJECT)
    if(CMAKE_CURRENT_SOURCE_DIR STREQUAL CMAKE_CURRENT_BINARY_DIR)
        message(AUTHOR_WARNING "Building in-place!")
    endif()
    set(WYKOBI_MASTER_PROJECT ON)
    message(STATUS "CMake ${CMAKE_VERSION}")
else()
    set(WYKOBI_MASTER_PROJECT OFF)
endif()

option(WYKOBI_INSTALL "Install wykobi header files?" ${WYKOBI_MASTER_PROJECT})
option(WYKOBI_TEST "Build wykobi test suite?" ${WYKOBI_MASTER_PROJECT})

set(WYKOBI_EXTERN_DIR ${CMAKE_CURRENT_SOURCE_DIR}/external)

set(WYKOBI_INCLUDE_DIR
    "${CMAKE_CURRENT_LIST_DIR}/include"
    CACHE INTERNAL "Location of wykobi headers")

add_library(${PROJECT_NAME} INTERFACE)
add_library(${PROJECT_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})
target_compile_features(${PROJECT_NAME} INTERFACE cxx_std_11)

target_include_directories(${PROJECT_NAME} INTERFACE
    $<BUILD_INTERFACE:${WYKOBI_INCLUDE_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)

if(WYKOBI_INSTALL)

    install(TARGETS ${PROJECT_NAME}
        EXPORT ${PROJECT_NAME}_Targets
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

    configure_package_config_file(
        ${PROJECT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in
        "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
        INSTALL_DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}/cmake)

    write_basic_package_version_file(
        ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake
        VERSION ${PROJECT_VERSION}
        COMPATIBILITY SameMajorVersion)

    install(EXPORT ${PROJECT_NAME}_Targets
        FILE ${PROJECT_NAME}Targets.cmake
        NAMESPACE ${PROJECT_NAME}::
        DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}/cmake)

    install(FILES "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
        "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
        DESTINATION ${CMAKE_INSTALL_DATAROOTDIR}/${PROJECT_NAME}/cmake)

    install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/${PROJECT_NAME} DESTINATION include)

endif()

if(WYKOBI_TEST)
    add_subdirectory(${WYKOBI_EXTERN_DIR}/googletest)
    add_subdirectory(tests)
endif()

if(NOT WYKOBI_MASTER_PROJECT)
    set(WYKOBI_FOUND TRUE
        CACHE INTERNAL "True if wykobi found")
endif()
