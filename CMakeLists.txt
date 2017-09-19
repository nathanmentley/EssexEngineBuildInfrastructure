list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/src/Daemons/ConfigDaemon/cmake/)

function(ee_setup_project ee_project_name ee_proj_directory)
    list(APPEND CMAKE_MODULE_PATH ${ee_proj_directory}/cmake/)

    #files

    file(GLOB ${ee_project_name}_HEADERS
        "${ee_proj_directory}/src/*.h"
        "${ee_proj_directory}/src/**/*.h"
    )
    file(GLOB ${ee_project_name}_SRC
        ${${ee_project_name}_HEADERS}
        "${ee_proj_directory}/src/*.cpp"
        "${ee_proj_directory}/src/**.cpp"
    )

    #setup headers

    file(COPY ${${ee_project_name}_HEADERS} DESTINATION "${CMAKE_BINARY_DIR}/lib/include/${ee_project_name}/")

    #setup build

    add_library(${ee_project_name} SHARED ${${ee_project_name}_SRC})
    #target_link_libraries(${ee_project_name} EssexEngineCore)

    #setup tests

    find_package(CATCH REQUIRED)

    file(GLOB ${ee_project_name}_TESTSRC
        "${ee_proj_directory}/test/*.cpp"
    )
    add_executable(${ee_project_name}Tests ${${ee_project_name}_TESTSRC})
    add_test(NAME ${ee_project_name}Tests COMMAND ${ee_project_name}Tests)

    target_include_directories(${ee_project_name}Tests PRIVATE ${CATCH_INCLUDE_DIRS})
endfunction(ee_setup_project)
