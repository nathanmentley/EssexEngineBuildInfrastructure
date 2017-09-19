list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/src/System/BuildInfrastructure/Modules/)

function(ee_setup_base_project ee_project_name ee_proj_directory)
    #files
    file(GLOB ${ee_project_name}_HEADERS
        "${ee_proj_directory}/src/*.h"
        "${ee_proj_directory}/src/*/*.h"
        "${ee_proj_directory}/src/*/*/*.h"
    )
    file(GLOB ${ee_project_name}_SRC
        ${${ee_project_name}_HEADERS}
        "${ee_proj_directory}/src/*.cpp"
        "${ee_proj_directory}/src/*/*.cpp"
        "${ee_proj_directory}/src/*/*/*.cpp"
    )

    #setup headers

    file(COPY ${${ee_project_name}_HEADERS} DESTINATION "${CMAKE_BINARY_DIR}/lib/include/${ee_project_name}/")

    #setup tests

    find_package(CATCH REQUIRED)

    file(GLOB ${ee_project_name}_TESTSRC
        "${ee_proj_directory}/test/*.cpp"
    )
    add_executable(${ee_project_name}Tests ${${ee_project_name}_TESTSRC})
    add_test(NAME ${ee_project_name}Tests COMMAND ${ee_project_name}Tests)

    target_include_directories(${ee_project_name}Tests PRIVATE ${CATCH_INCLUDE_DIRS})

    set(${ee_project_name}_SRC ${${ee_project_name}_SRC} PARENT_SCOPE)
endfunction(ee_setup_base_project)

function(ee_setup_project ee_project_name ee_proj_directory)
    ee_setup_base_project(${ee_project_name} ${ee_proj_directory})
    
    add_library(${ee_project_name} SHARED ${${ee_project_name}_SRC})
endfunction(ee_setup_project)

function(ee_setup_bootloader ee_project_name ee_proj_directory)
    ee_setup_base_project(${ee_project_name} ${ee_proj_directory})
    
    add_executable(${ee_project_name} ${${ee_project_name}_SRC})
endfunction(ee_setup_bootloader)
