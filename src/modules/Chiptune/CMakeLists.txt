cmake_minimum_required(VERSION 3.12)
project(Chiptune)

set(Chiptune_HDR
    Chiptune.hpp
    Common.hpp
)

set(Chiptune_SRC
    Chiptune.cpp
    Common.cpp
)

set(Chiptune_RESOURCES
    icons.qrc
)

set(Chiptune_LIBS
)

if(USE_CHIPTUNE_GME)
    find_package(GME REQUIRED)
    list(APPEND Chiptune_HDR GME.hpp)
    list(APPEND Chiptune_SRC GME.cpp)
    add_definitions(-DUSE_GME)
    if(GME_HAS_EFFECTS)
        add_definitions(-DUSE_GME_EFFECTS)
    endif()
    include_directories(${LIBGME_INCLUDE_DIRS})
    link_directories(${LIBGME_LIBRARY_DIRS})
    list(APPEND Chiptune_LIBS ${LIBGME_LIBRARIES})
endif()

if(USE_CHIPTUNE_SID)
    pkg_check_modules(LIBSIDPLAYFP REQUIRED libsidplayfp)
    list(APPEND Chiptune_HDR SIDPlay.hpp)
    list(APPEND Chiptune_SRC SIDPlay.cpp)
    add_definitions(-DUSE_SIDPLAY)
    include_directories(${LIBSIDPLAYFP_INCLUDE_DIRS})
    link_directories(${LIBSIDPLAYFP_LIBRARY_DIRS})
    list(APPEND Chiptune_LIBS ${LIBSIDPLAYFP_LIBRARIES})
endif()

add_library(${PROJECT_NAME} MODULE
    ${Chiptune_HDR}
    ${Chiptune_SRC}
    ${Chiptune_RESOURCES}
)

if(USE_PCH)
    target_precompile_headers(${PROJECT_NAME} PRIVATE
        ${Chiptune_HDR}
    )
endif()

target_link_libraries(${PROJECT_NAME}
    libqmplay2
)

if(WIN32)
    set(CUSTOM_CHIPTUNE_LIBRARIES "" CACHE STRING "Custom libraries for Chiptune")
endif()

if(NOT WIN32 OR CMAKE_HOST_WIN32)
    target_link_libraries(${PROJECT_NAME} ${Chiptune_LIBS})
endif()

if(WIN32 AND CUSTOM_CHIPTUNE_LIBRARIES)
    separate_arguments(CUSTOM_CHIPTUNE_LIBRARIES)
    target_link_libraries(${PROJECT_NAME} ${CUSTOM_CHIPTUNE_LIBRARIES})
endif()

install(TARGETS ${PROJECT_NAME} LIBRARY DESTINATION ${MODULES_INSTALL_PATH})
