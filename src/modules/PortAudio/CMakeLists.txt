cmake_minimum_required(VERSION 3.12)
project(PortAudio)

set(PortAudio_HDR
    PortAudio.hpp
    PortAudioCommon.hpp
    PortAudioWriter.hpp
)

set(PortAudio_SRC
    PortAudio.cpp
    PortAudioCommon.cpp
    PortAudioWriter.cpp
)

if(APPLE)
    list(APPEND PortAudio_SRC
        3rdparty/CoreAudio/AudioDevice.mm
        3rdparty/CoreAudio/AudioDeviceList.cpp
    )
endif()

set(PortAudio_RESOURCES
    icon.qrc
)

pkg_check_modules(LIBPORTAUDIO portaudio-2.0 REQUIRED)
link_directories(${LIBPORTAUDIO_LIBRARY_DIRS})

include_directories(${LIBPORTAUDIO_INCLUDE_DIRS})

add_library(${PROJECT_NAME} MODULE
    ${PortAudio_HDR}
    ${PortAudio_SRC}
    ${PortAudio_RESOURCES}
)

if(USE_PCH)
    target_precompile_headers(${PROJECT_NAME}
        PRIVATE
        ${PortAudio_HDR}
    )
endif()

target_link_libraries(${PROJECT_NAME}
    libqmplay2
)

if(WIN32)
    target_compile_definitions(${PROJECT_NAME} PRIVATE -DINITGUID)
    set(CUSTOM_PORTAUDIO_LIBRARIES "" CACHE STRING "Custom libraries for PortAudio")
endif()

if(NOT WIN32 OR CMAKE_HOST_WIN32)
    target_link_libraries(${PROJECT_NAME} ${LIBPORTAUDIO_LIBRARIES})
endif()

if(WIN32 AND CUSTOM_PORTAUDIO_LIBRARIES)
    separate_arguments(CUSTOM_PORTAUDIO_LIBRARIES)
    target_link_libraries(${PROJECT_NAME} ${CUSTOM_PORTAUDIO_LIBRARIES})
elseif(APPLE)
    target_link_libraries(${PROJECT_NAME} "-framework CoreAudio -framework Foundation")
endif()

install(TARGETS ${PROJECT_NAME} LIBRARY DESTINATION ${MODULES_INSTALL_PATH})
