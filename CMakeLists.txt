set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -g -std=c++17 -stdlib=libc++")
#set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -Wall")
#set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -Wall -O3")

list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/src/System/BuildInfrastructure/Modules/)

function (ee_clone_headers ee_project_name ee_proj_directory ee_header_target_dir)
    # Scan through resource folder for updated files and copy if none existing or changed
    file(GLOB ${ee_project_name}_HEADERS
        "${ee_proj_directory}/src/*.h"
        "${ee_proj_directory}/src/*/*.h"
        "${ee_proj_directory}/src/*/*/*.h"
    )

    foreach(resource ${${ee_project_name}_HEADERS})
        get_filename_component(filename ${resource} NAME)
        set(output "${ee_header_target_dir}/${filename}")
        
        add_custom_command(
            COMMENT "Moving updated resource-file '${filename}'"
            OUTPUT ${output}
            DEPENDS ${resource}
            COMMAND ${CMAKE_COMMAND} -E copy_if_different
            ${resource}
            ${output}
        )
        
        add_custom_target("${ee_project_name}_${filename}" ALL DEPENDS ${resource} ${output})
    endforeach()
    set(${ee_project_name}_HEADERS ${${ee_project_name}_HEADERS} PARENT_SCOPE)
endfunction(ee_clone_headers)

function(ee_setup_base_project ee_project_name ee_proj_directory)
    #setup headers
    ee_clone_headers("${ee_project_name}" "${ee_proj_directory}" "${CMAKE_BINARY_DIR}/lib/include/${ee_project_name}")

    #files
    file(GLOB ${ee_project_name}_SRC
        ${${ee_project_name}_HEADERS}
        "${ee_proj_directory}/src/*.cpp"
        "${ee_proj_directory}/src/*/*.cpp"
        "${ee_proj_directory}/src/*/*/*.cpp"
    )

    
    #setup tests
    find_package(CATCH REQUIRED)

    file(GLOB ${ee_project_name}_TESTSRC
        "${ee_proj_directory}/test/*.cpp"
    )
    
    add_executable(${ee_project_name}Tests ${${ee_project_name}_TESTSRC})
    add_test(NAME ${ee_project_name}Tests COMMAND ${ee_project_name}Tests)

    target_include_directories(${ee_project_name}Tests PRIVATE ${CATCH_INCLUDE_DIRS})
    foreach(resource ${${ee_project_name}_HEADERS})
        get_filename_component(filename ${resource} NAME)
        add_dependencies(${ee_project_name}Tests "${ee_project_name}_${filename}")
    endforeach()

    set(${ee_project_name}_HEADERS ${${ee_project_name}_HEADERS} PARENT_SCOPE)
    set(${ee_project_name}_SRC ${${ee_project_name}_SRC} PARENT_SCOPE)
endfunction(ee_setup_base_project)

function(ee_setup_project ee_project_name ee_proj_directory)
    ee_setup_base_project(${ee_project_name} ${ee_proj_directory})

    add_library(${ee_project_name} SHARED ${${ee_project_name}_SRC})
    foreach(resource ${${ee_project_name}_HEADERS})
        get_filename_component(filename ${resource} NAME)
        add_dependencies(${ee_project_name} "${ee_project_name}_${filename}")
    endforeach()
endfunction(ee_setup_project)

function(ee_setup_bootloader ee_project_name ee_proj_directory)
    ee_setup_base_project(${ee_project_name} ${ee_proj_directory})
    
    add_executable(${ee_project_name} ${${ee_project_name}_SRC})
    foreach(resource ${${ee_project_name}_HEADERS})
        get_filename_component(filename ${resource} NAME)
        add_dependencies(${ee_project_name} "${ee_project_name}_${filename}")
    endforeach()
endfunction(ee_setup_bootloader)

find_package(Doxygen REQUIRED)
if (DOXYGEN_FOUND)
  message("Doxygen is installed")
else (DOXYGEN_FOUND)
  message("Doxygen need to be installed to generate the doxygen documentation")
endif (DOXYGEN_FOUND)
