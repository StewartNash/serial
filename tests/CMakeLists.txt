find_package(ament_cmake_gtest REQUIRED)
    
if(UNIX)
    ament_add_gtest(${PROJECT_NAME}-test unix_serial_tests.cc)
    target_link_libraries(${PROJECT_NAME}-test ${PROJECT_NAME})
    if(NOT APPLE)
        target_link_libraries(${PROJECT_NAME}-test util)
    endif()

    if(NOT APPLE)  # these tests are unreliable on macOS
      ament_add_gtest(${PROJECT_NAME}-test-timer unit/unix_timer_tests.cc)
      target_link_libraries(${PROJECT_NAME}-test-timer ${PROJECT_NAME})
    endif()
endif()
