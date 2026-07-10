cmake_minimum_required(VERSION 3.21)
project(NorthLauncher VERSION 1.0.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)
set(CMAKE_AUTOUIC ON)

if(WIN32)
    add_definitions(-D_WIN32_WINNT=0x0A00)
endif()

find_package(Qt6 REQUIRED COMPONENTS Widgets Network Concurrent)
find_package(CURL REQUIRED)
find_package(nlohmann_json REQUIRED)
find_package(OpenSSL REQUIRED)
find_package(libzip REQUIRED)

set(SOURCES
    src/main.cpp
    src/UI/MainWindow.cpp
    src/UI/MainWindow.h
    src/Auth/MicrosoftAuth.cpp
    src/Auth/MicrosoftAuth.h
    src/Launcher/VersionManager.cpp
    src/Launcher/VersionManager.h
    src/Launcher/Downloader.cpp
    src/Launcher/Downloader.h
    src/Launcher/GameLauncher.cpp
    src/Launcher/GameLauncher.h
    src/Utils/Http.cpp
    src/Utils/Http.h
    src/Utils/Paths.cpp
    src/Utils/Paths.h
)

add_executable(NorthLauncher WIN32 ${SOURCES})

target_include_directories(NorthLauncher PRIVATE src)

target_link_libraries(NorthLauncher PRIVATE
    Qt6::Widgets
    Qt6::Network
    Qt6::Concurrent
    CURL::libcurl
    nlohmann_json::nlohmann_json
    OpenSSL::Crypto
    libzip::zip
)

# Copy Qt runtime DLLs next to the exe automatically on Windows (requires windeployqt in PATH)
if(WIN32)
    find_program(WINDEPLOYQT_EXECUTABLE windeployqt HINTS "${Qt6_DIR}/../../../bin")
    if(WINDEPLOYQT_EXECUTABLE)
        add_custom_command(TARGET NorthLauncher POST_BUILD
            COMMAND "${WINDEPLOYQT_EXECUTABLE}" --release "$<TARGET_FILE:NorthLauncher>"
            COMMENT "Running windeployqt to bundle Qt DLLs"
        )
    endif()
endif()

install(TARGETS NorthLauncher RUNTIME DESTINATION .)
