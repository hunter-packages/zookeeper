CMAKE_MINIMUM_REQUIRED(VERSION 2.8)

PROJECT(zookeeper)

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
else()
    set(ZK_LINK_LIBS ${ZK_LINK_LIBS} m)
endif()


set(BUILD_DYNAMIC OFF CACHE BOOL "build dynamic libs instead of static ones")

if(BUILD_DYNAMIC)
    set(ZK_BUILD_TYPE SHARED)
else()
    set(ZK_BUILD_TYPE STATIC)
endif()

include_directories(include/ generated/ ./)

######### ST ###########   
add_library(zookeeper_st ${ZK_BUILD_TYPE}
    ${COMMON_SRCS}
    src/st_adaptor.c
)
target_link_libraries(zookeeper_st ${ZK_LINK_LIBS})

add_executable(cli_st
    src/cli.c
)
target_link_libraries(cli_st zookeeper_st)

######### MT ###########   
add_library(zookeeper_mt ${ZK_BUILD_TYPE}
    ${COMMON_SRCS}
    src/mt_adaptor.c
    src/load_gen.c    
)
find_package(Threads)
target_link_libraries(zookeeper_mt ${ZK_LINK_LIBS} ${CMAKE_THREAD_LIBS_INIT})
target_compile_definitions(zookeeper_mt PRIVATE THREADED)

add_executable(cli_mt
    src/cli.c
)
target_link_libraries(cli_mt zookeeper_mt)
