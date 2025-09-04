cmake_minimum_required(VERSION 3.12)

project(LuaSec LANGUAGES C)

set(OPENSSL_ROOT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/prebuilt-ssl/arm64-v8a CACHE PATH "Путь к prebuilt OpenSSL для ARM64-v8A")

find_library(OPENSSL_SSL_LIBRARY NAMES ssl libssl.a HINTS ${OPENSSL_ROOT_DIR}/lib)
find_library(OPENSSL_CRYPTO_LIBRARY NAMES crypto libcrypto.a HINTS ${OPENSSL_ROOT_DIR}/lib)
find_path(OPENSSL_INCLUDE_DIR NAMES openssl/ssl.h HINTS ${OPENSSL_ROOT_DIR}/include)

if(NOT OPENSSL_INCLUDE_DIR OR NOT OPENSSL_SSL_LIBRARY OR NOT OPENSSL_CRYPTO_LIBRARY)
    message(FATAL_ERROR "Не удалось найти OpenSSL в ${OPENSSL_ROOT_DIR}. Проверьте путь.")
endif()

message(STATUS "Найдены заголовки OpenSSL: ${OPENSSL_INCLUDE_DIR}")
message(STATUS "Найдена библиотека OpenSSL SSL: ${OPENSSL_SSL_LIBRARY}")
message(STATUS "Найдена библиотека OpenSSL Crypto: ${OPENSSL_CRYPTO_LIBRARY}")


set(LUA_INCLUDE_DIR ${LUAJIT_INCLUDE_DIR} CACHE PATH "Путь к заголовочным файлам Lua")


set(LUASEC_SOURCES
    src/options.c
    src/config.c
    src/ec.c
    src/x509.c
    src/context.c
    src/ssl.c
    src/luasocket/buffer.c
    src/luasocket/io.c
    src/luasocket/timeout.c
    src/luasocket/usocket.c
)

add_library(luasec STATIC ${LUASEC_SOURCES})


target_include_directories(luasec
    PUBLIC
        ${CMAKE_CURRENT_SOURCE_DIR}/src
    PRIVATE
        ${CMAKE_CURRENT_SOURCE_DIR}/src/luasocket
        ${OPENSSL_INCLUDE_DIR}
        ${LUA_INCLUDE_DIR}
)

target_compile_definitions(luasec PRIVATE
    WITH_LUASOCKET
    LUASOCKET_DEBUG
    OPENSSL_API_COMPAT=0x10101000L
)

target_link_libraries(luasec PRIVATE
    ${OPENSSL_SSL_LIBRARY}
    ${OPENSSL_CRYPTO_LIBRARY}
)