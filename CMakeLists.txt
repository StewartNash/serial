cmake_minimum_required(VERSION 3.8)
project(serial)

# Find ament
find_package(ament_cmake REQUIRED)

# Apple-specific frameworks
if(APPLE)
    find_library(IOKIT_LIBRARY IOKit)
    find_library(FOUNDATION_LIBRARY Foundation)
endif()

# Common sources
set(serial_SRCS
    src/serial.cc
    include/serial/serial.h
    include/serial/v8stdint.h
)

# Platform-specific sources
if(APPLE)
    list(APPEND serial_SRCS
        src/impl/unix.cc
        src/impl/list_ports/list_ports_osx.cc
    )
elseif(UNIX)
    list(APPEND serial_SRCS
        src/impl/unix.cc
        src/impl/list_ports/list_ports_linux.cc
    )
else()
    list(APPEND serial_SRCS
        src/impl/win.cc
        src/impl/list_ports/list_ports_win.cc
    )
endif()

# Define library
add_library(${PROJECT_NAME} ${serial_SRCS})
target_include_directories(${PROJECT_NAME} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

# Link system libraries
if(APPLE)
    target_link_libraries(${PROJECT_NAME} ${FOUNDATION_LIBRARY} ${IOKIT_LIBRARY})
elseif(UNIX)
    target_link_libraries(${PROJECT_NAME} rt pthread)
else()
    target_link_libraries(${PROJECT_NAME} setupapi)
endif()

# Optionally build example
add_executable(serial_example examples/serial_example.cc)
target_link_libraries(serial_example ${PROJECT_NAME})

# Install
install(TARGETS ${PROJECT_NAME}
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin
)

install(DIRECTORY include/ DESTINATION include)

# ament exports
ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME})
ament_package()
