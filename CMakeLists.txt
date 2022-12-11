CMAKE_MINIMUM_REQUIRED(VERSION 3.23)

#set(CMAKE_OSX_DEPLOYMENT_TARGET "10.15" CACHE STRING "Minimum OS X deployment version")

PROJECT(
  PixelGameEngine
  VERSION 2.21
  DESCRIPTION "The official distribution of olcPixelGameEngine, a tool used in javidx9's YouTube videos and projects."
  LANGUAGES CXX
)

if (CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
    set(PGE_STANDALONE TRUE)
endif()

SET(PGE_SOURCE_DIR ${PROJECT_SOURCE_DIR})

SET(LIBRARY_TARGET_NAME ${PROJECT_NAME})

SET(LIBRARY_TARGET_NAME_EXTEN ${PROJECT_NAME}Extensions)

SET(LIBRARY_TARGET_NAME_UTILS ${PROJECT_NAME}Utilities)

if (PGE_STANDALONE)
  SET(PGE_BUILD_EXTRAS FALSE)
else()
  SET(PGE_BUILD_EXTRAS TRUE)
endif()

option(PGE_BUILD_EXAMPLES "Build the PGE example programs" ${PGE_BUILD_EXTRAS})
option(PGE_INSTALL "Generate installation target" ${PGE_STANDALONE})

SET(LINKER_LANGUAGE CXX)
SET(CMAKE_CXX_STANDARD 20)
SET(CMAKE_CXX_STANDARD_REQUIRED ON)
SET(CMAKE_CXX_EXTENSIONS ON)
SET(THREADS_PREFER_PTHREAD_FLAD ON)

SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib)
SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib)
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)

ADD_LIBRARY(${LIBRARY_TARGET_NAME} INTERFACE)
ADD_LIBRARY(${LIBRARY_TARGET_NAME_EXTEN} INTERFACE)
ADD_LIBRARY(${LIBRARY_TARGET_NAME_UTILS} INTERFACE)

TARGET_SOURCES(${LIBRARY_TARGET_NAME}       INTERFACE FILE_SET HEADERS
                                            BASE_DIRS include
                                            FILES include/olcPixelGameEngine.hpp
)
TARGET_SOURCES(${LIBRARY_TARGET_NAME_EXTEN} INTERFACE FILE_SET HEADERS
                                            BASE_DIRS extensions
                                            FILES extensions/olcPGEX_Graphics2D.hpp
                                                  extensions/olcPGEX_Graphics3D.hpp
                                                  extensions/olcPGEX_Network.hpp
                                                  extensions/olcPGEX_PopUpMenu.hpp
                                                  extensions/olcPGEX_QuickGUI.hpp
                                                  extensions/olcPGEX_RayCastWorld.hpp
                                                  extensions/olcPGEX_Sound.hpp
                                                  extensions/olcPGEX_SplashScreen.hpp
                                                  extensions/olcPGEX_TransformedView.hpp
                                                  extensions/olcPGEX_Wireframe.hpp
)
TARGET_SOURCES(${LIBRARY_TARGET_NAME_UTILS} INTERFACE FILE_SET HEADERS
                                            BASE_DIRS utilities
                                            FILES utilities/olcUTIL_Animate2D.hpp
                                                  utilities/olcUTIL_Camera2D.hpp
                                                  utilities/olcUTIL_Container.hpp
                                                  utilities/olcUTIL_DataFile.hpp
                                                  utilities/olcUTIL_Geometry2D.hpp
                                                  utilities/olcUTIL_Palette.hpp
                                                  utilities/olcUTIL_QuadTree.hpp
)

SET(PGE_HEADERS)
LIST(APPEND PGE_HEADERS
  ${LIBRARY_TARGET_NAME}
  ${LIBRARY_TARGET_NAME_EXTEN}
  ${LIBRARY_TARGET_NAME_UTILS}
)

SET_TARGET_PROPERTIES(${PGE_HEADERS}    PROPERTIES VERSION  "${${PROJECT_NAME}_VERSION}"
                                           LINKER_LANGUAGE  "${LINKER_LANGUAGE}"
                                        CMAKE_CXX_STANDARD  "${CMAKE_CXX_STANDARD}"
                               CMAKE_CXX_STANDARD_REQUIRED  "${CMAKE_CXX_STANDARD_REQUIRED}"
                                      CMAKE_CXX_EXTENSIONS  "${CMAKE_CXX_EXTENSIONS}"
                               THREADS_PREFER_PTHREAD_FLAD  "${THREADS_PREFER_PTHREAD_FLAD}"
)

if(NOT CMAKE_CONFIGURATION_TYPES)
    if(NOT CMAKE_BUILD_TYPE)
        message(STATUS "Setting build type to 'Release' as none was specified.")
        set_property(CACHE CMAKE_BUILD_TYPE PROPERTY VALUE "Release")
    endif()
endif()

#region Thirdparty Libraries

SET(PGE_LIB)  # Link against
SET(PGE_INCLUDE_DIR)   # Include Directories

# Threads
FIND_PACKAGE(Threads REQUIRED)
FIND_PACKAGE(OpenGL REQUIRED)

LIST(APPEND PGE_LIB Threads::Threads)

# OpenGL
if (LINUX OR WIN32)
  LIST(APPEND PGE_LIB
    OpenGL::OpenGL
    OpenGL::GL
  #  OpenGL::GLU
    OpenGL::GLX
  #  OpenGL::EGL
  )
endif(LINUX OR WIN32)
if (APPLE)
  LIST(APPEND PGE_LIB ${OPENGL_LIBRARIES})
endif(APPLE)

if (LINUX OR APPLE)
  # X11
  FIND_PACKAGE(X11 REQUIRED)
  # PNG
  FIND_PACKAGE(PNG REQUIRED)
  # X11
  if(X11_FOUND)
    LIST(APPEND PGE_LIB ${X11_LIBRARIES})
  endif(X11_FOUND)

  # PNG
  LIST(APPEND PGE_INCLUDE_DIR ${PNG_INCLUDE_DIRS})
  LIST(APPEND PGE_LIB PNG::PNG)
endif()

if (APPLE)
  # Carbon
  FIND_LIBRARY(CARBON_LIBRARY Carbon REQUIRED)
  # GLUT (OpenGL Utilities)
  FIND_PACKAGE(GLUT REQUIRED)

  LIST(APPEND PGE_LIB GLUT::GLUT)
endif(APPLE)

if(WIN32)
  # Win32 SDK
  #INCLUDE_DIRECTORIES(${WinSDK})
  LIST(APPEND PGE_INCLUDE_DIR ${WinSDK})
endif(WIN32)

LIST(REMOVE_DUPLICATES PGE_INCLUDE_DIR)

TARGET_LINK_DIRECTORIES(${LIBRARY_TARGET_NAME} INTERFACE ${PGE_INCLUDE_DIR})
TARGET_LINK_DIRECTORIES(${LIBRARY_TARGET_NAME_EXTEN} INTERFACE ${PGE_INCLUDE_DIR})
TARGET_LINK_DIRECTORIES(${LIBRARY_TARGET_NAME_UTILS} INTERFACE ${PGE_INCLUDE_DIR})

TARGET_INCLUDE_DIRECTORIES(${LIBRARY_TARGET_NAME} INTERFACE ${PGE_INCLUDE_DIR})
TARGET_INCLUDE_DIRECTORIES(${LIBRARY_TARGET_NAME_EXTEN} INTERFACE ${PGE_INCLUDE_DIR})
TARGET_INCLUDE_DIRECTORIES(${LIBRARY_TARGET_NAME_UTILS} INTERFACE ${PGE_INCLUDE_DIR})

TARGET_LINK_LIBRARIES(${LIBRARY_TARGET_NAME} INTERFACE ${PGE_LIB})
TARGET_LINK_LIBRARIES(${LIBRARY_TARGET_NAME_EXTEN} INTERFACE ${LIBRARY_TARGET_NAME})
TARGET_LINK_LIBRARIES(${LIBRARY_TARGET_NAME_UTILS} INTERFACE ${LIBRARY_TARGET_NAME})

#endregion Thirdparty Libraries

ADD_LIBRARY(${PROJECT_NAME}::${LIBRARY_TARGET_NAME} ALIAS ${LIBRARY_TARGET_NAME})
ADD_LIBRARY(${PROJECT_NAME}::Extensions ALIAS ${LIBRARY_TARGET_NAME_EXTEN})
ADD_LIBRARY(${PROJECT_NAME}::Utilities ALIAS ${LIBRARY_TARGET_NAME_UTILS})

if (PGE_BUILD_EXAMPLES)
  add_subdirectory(examples)
endif()

if (PGE_INSTALL)
  include(GNUInstallDirs)
  SET(PGE_CMAKE_PATH "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}")

  INSTALL(TARGETS ${LIBRARY_TARGET_NAME}
        EXPORT                        "${PROJECT_NAME}Targets"
        LIBRARY DESTINATION           "${CMAKE_INSTALL_LIBDIR}/${PROJECT_NAME}"
        ARCHIVE DESTINATION           "${CMAKE_INSTALL_LIBDIR}/${PROJECT_NAME}"
        PRIVATE_HEADER DESTINATION    "${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}"
        PUBLIC_HEADER DESTINATION     "${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}"
        FILE_SET HEADERS DESTINATION  "${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}"
  )
  INSTALL(TARGETS ${LIBRARY_TARGET_NAME_EXTEN}
        EXPORT                        "${PROJECT_NAME}Targets"
        LIBRARY DESTINATION           "${CMAKE_INSTALL_LIBDIR}/${PROJECT_NAME}/extensions"
        ARCHIVE DESTINATION           "${CMAKE_INSTALL_LIBDIR}/${PROJECT_NAME}/extensions"
        PRIVATE_HEADER DESTINATION    "${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}/extensions"
        PUBLIC_HEADER DESTINATION     "${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}/extensions"
        FILE_SET HEADERS DESTINATION  "${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}/extensions"
  )
  INSTALL(TARGETS ${LIBRARY_TARGET_NAME_UTILS}
        EXPORT                        "${PROJECT_NAME}Targets"
        LIBRARY DESTINATION           "${CMAKE_INSTALL_LIBDIR}/${PROJECT_NAME}/utilities"
        ARCHIVE DESTINATION           "${CMAKE_INSTALL_LIBDIR}/${PROJECT_NAME}/utilities"
        PRIVATE_HEADER DESTINATION    "${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}/utilities"
        PUBLIC_HEADER DESTINATION     "${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}/utilities"
        FILE_SET HEADERS DESTINATION  "${CMAKE_INSTALL_INCLUDEDIR}/${PROJECT_NAME}/utilities"
  )
  INSTALL(EXPORT      "${PROJECT_NAME}Targets"
        DESTINATION   "${PGE_CMAKE_PATH}"
        NAMESPACE     "${PROJECT_NAME}::"
        FILE          "${PROJECT_NAME}Targets.cmake"
  )

  INCLUDE(CMakePackageConfigHelpers)

  SET(CONF_INCLUDE_DIRS "${PROJECT_SOURCE_DIR}/include" "${PROJECT_BINARY_DIR}/include")

  CONFIGURE_FILE("${PROJECT_SOURCE_DIR}/CMake/Modules.cmake.in" "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Modules.cmake" @ONLY)

  CONFIGURE_PACKAGE_CONFIG_FILE("${PROJECT_SOURCE_DIR}/CMake/Config.cmake.in"
    "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
    INSTALL_DESTINATION "${PGE_CMAKE_PATH}"
    PATH_VARS
      PROJECT_NAME CMAKE_INSTALL_LIBDIR CMAKE_INSTALL_INCLUDEDIR CONF_INCLUDE_DIRS
  )
  WRITE_BASIC_PACKAGE_VERSION_FILE("${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
    VERSION ${${PROJECT_NAME_UC}_VERSION}
    COMPATIBILITY SameMajorVersion
    ARCH_INDEPENDENT
  )

  INSTALL(
    FILES "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
          "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
          "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Modules.cmake"
    DESTINATION "${PGE_CMAKE_PATH}"
  )
endif(PGE_INSTALL)


EXPORT(TARGETS  ${PGE_HEADERS}
  NAMESPACE     "${PROJECT_NAME}::"
  FILE          "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Targets.cmake"
  EXPORT_LINK_INTERFACE_LIBRARIES)
