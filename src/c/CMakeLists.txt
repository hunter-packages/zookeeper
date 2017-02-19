CMAKE_MINIMUM_REQUIRED(VERSION 3.0)

PROJECT(zookeeper VERSION 3.4.9)

set(COMMON_SRCS
    src/recordio.c
    src/zk_adaptor.h
    src/zk_hashtable.h
    src/zookeeper.c
    src/zk_hashtable.c
    src/zk_log.c

    src/hashtable/hashtable.c
    src/hashtable/hashtable.h
    src/hashtable/hashtable_itr.c
    src/hashtable/hashtable_itr.h
    src/hashtable/hashtable_private.h

    include/proto.h
    include/recordio.h
    include/winconfig.h
    include/winstdint.h
    include/zookeeper.h
    include/zookeeper_log.h
    include/zookeeper_version.h

    generated/zookeeper.jute.c
    generated/zookeeper.jute.h
)

set(ZK_LINK_LIBS "")

if(WIN32)
    set(COMMON_SRCS
        ${COMMON_SRCS}
        src/winport.h
        src/winport.c
    )
    set(ZK_LINK_LIBS wsock32 ws2_32)
else()
    set(ZK_LINK_LIBS ${ZK_LINK_LIBS} m)
endif()


set(BUILD_DYNAMIC OFF CACHE BOOL "build dynamic libs instead of static ones")
set(BUILD_CLIS OFF CACHE BOOL "build example command line programs")


if(BUILD_DYNAMIC)
    set(ZK_BUILD_TYPE SHARED)
    set(ZK_DEFINES DLL_EXPORT)
    set(ZK_INTERFACE_DEFINES "")
else()
    set(ZK_BUILD_TYPE STATIC)
    set(ZK_DEFINES USE_STATIC_LIB)
    set(ZK_INTERFACE_DEFINES USE_STATIC_LIB)
endif()

include_directories(include/ generated/ ./)

######### ST ###########   
add_library(zookeeper_st ${ZK_BUILD_TYPE}
    ${COMMON_SRCS}
    src/st_adaptor.c
)
target_link_libraries(zookeeper_st ${ZK_LINK_LIBS})
target_compile_definitions(
    zookeeper_st 
    PRIVATE ${ZK_DEFINES}
    INTERFACE ${ZK_INTERFACE_DEFINES}
)

if(BUILD_CLIS)
    add_executable(cli_st
        src/cli.c
    )
    target_link_libraries(cli_st zookeeper_st)
endif()

######### MT ###########   
add_library(zookeeper_mt ${ZK_BUILD_TYPE}
    ${COMMON_SRCS}
    src/mt_adaptor.c   
)
find_package(Threads)
target_link_libraries(zookeeper_mt ${ZK_LINK_LIBS} ${CMAKE_THREAD_LIBS_INIT})
target_compile_definitions(
    zookeeper_mt 
    PRIVATE ${ZK_DEFINES} THREADED
    INTERFACE ${ZK_INTERFACE_DEFINES} THREADED
)

if(BUILD_CLIS)
    add_executable(cli_mt
        src/cli.c
    )
    target_link_libraries(cli_mt zookeeper_mt)
endif()



set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated/cmake")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)

# Note: PROJECT_VERSION is used as a VERSION
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

# Note: variable 'targets_export_name' used
configure_file("cmake/Config.cmake.in" "${project_config}" @ONLY)


install(
    TARGETS zookeeper_mt zookeeper_st
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)

file(GLOB include_files "include/*.h")
install(
    FILES ${include_files} generated/zookeeper.jute.h
    DESTINATION "${include_install_dir}"
)


install(
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)


install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)
