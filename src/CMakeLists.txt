
# call redis's `mkreleasehdr.sh` to Generate `release.h`
message(STATUS "Generating release.h")
execute_process(
        WORKING_DIRECTORY ${REDIS_ROOT}/src/
        COMMAND sh -c ./mkreleasehdr.sh
)

set (REDIS_SERVER_SRC
        acl.c adlist.c ae.c anet.c aof.c bio.c
        bitops.c blocked.c childinfo.c cluster.c config.c connection.c crc16.c crc64.c db.c debug.c
        defrag.c dict.c evict.c expire.c geo.c geohash.c geohash_helper.c gopher.c hyperloglog.c
        intset.c latency.c lazyfree.c listpack.c localtime.c lolwut.c lolwut5.c lolwut6.c lzf_c.c
        lzf_d.c memtest.c module.c multi.c networking.c notify.c object.c pqsort.c pubsub.c quicklist.c
        rand.c rax.c rdb.c redis-check-aof.c redis-check-rdb.c
        release.c replication.c rio.c scripting.c sds.c sentinel.c server.c setproctitle.c sha1.c
        sha256.c siphash.c slowlog.c sort.c sparkline.c syncio.c t_hash.c t_list.c t_set.c t_stream.c t_string.c
        t_zset.c tls.c tracking.c util.c ziplist.c zipmap.c zmalloc.c endianconv.c)

include_directories(redis-server
        ${REDIS_ROOT}/src
        ${REDIS_ROOT}/deps/hiredis
        ${REDIS_ROOT}/deps/linenoise
        ${REDIS_ROOT}/deps/lua/src
        )

link_directories(
        ${REDIS_ROOT}/deps/hiredis
        ${REDIS_ROOT}/deps/linenoise
        ${REDIS_ROOT}/deps/lua/src
)

add_executable(redis-server ${REDIS_SERVER_SRC})
add_executable(redis-cli anet.c adlist.c dict.c redis-cli.c zmalloc.c release.c anet.c ae.c crc64.c siphash.c crc16.c)
add_executable(redis-benchmark ae.c anet.c redis-benchmark.c adlist.c zmalloc.c)

target_link_libraries(redis-server hiredis lua m pthread dl sasl2)
target_link_libraries(redis-cli linenoise hiredis m pthread dl)
target_link_libraries(redis-benchmark hiredis m pthread dl)

add_custom_target(redis-sentinel ALL DEPENDS redis-server
        COMMAND cp ${PROJECT_BINARY_DIR}/redis-server ${PROJECT_BINARY_DIR}/redis-sentinel
        COMMAND cp ${PROJECT_BINARY_DIR}/redis-server ${PROJECT_BINARY_DIR}/redis-check-rdb
        COMMAND cp ${PROJECT_BINARY_DIR}/redis-server ${PROJECT_BINARY_DIR}/redis-check-aof
        )

add_custom_target(runtest DEPENDS redis-server
        COMMAND cp ${PROJECT_BINARY_DIR}/redis-server ${REDIS_ROOT}/src/redis-server
        COMMAND cp ${PROJECT_BINARY_DIR}/redis-server ${REDIS_ROOT}/src/redis-sentinel
        COMMAND cp ${PROJECT_BINARY_DIR}/redis-server ${REDIS_ROOT}/src/redis-check-rdb
        COMMAND cp ${PROJECT_BINARY_DIR}/redis-server ${REDIS_ROOT}/src/redis-check-aof
        COMMAND cd ${REDIS_ROOT} && sh ./runtest
        )

