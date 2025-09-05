cmake_minimum_required(VERSION 3.12)

project(LuaSec LANGUAGES C)

set(OPENSSL_ROOT_DIR ${CMAKE_CURRENT_SOURCE_DIR}/prebuilt-ssl/arm64-v8a CACHE PATH "Путь к prebuilt OpenSSL для ARM64-v8A")

set(OPENSSL_INCLUDE_DIR ${OPENSSL_ROOT_DIR}/include)
set(OPENSSL_SSL_LIBRARY ${OPENSSL_ROOT_DIR}/lib/libssl.a)
set(OPENSSL_CRYPTO_LIBRARY ${OPENSSL_ROOT_DIR}/lib/libcrypto.a)

if(NOT EXISTS ${OPENSSL_INCLUDE_DIR}/openssl/ssl.h)
    message(FATAL_ERROR "Не найден заголовочный файл OpenSSL: ${OPENSSL_INCLUDE_DIR}/openssl/ssl.h")
endif()

if(NOT EXISTS ${OPENSSL_SSL_LIBRARY})
    message(FATAL_ERROR "Не найдена библиотека SSL: ${OPENSSL_SSL_LIBRARY}")
endif()

if(NOT EXISTS ${OPENSSL_CRYPTO_LIBRARY})
    message(FATAL_ERROR "Не найдена библиотека Crypto: ${OPENSSL_CRYPTO_LIBRARY}")
endif()

message(STATUS "Используются заголовки OpenSSL: ${OPENSSL_INCLUDE_DIR}")
message(STATUS "Используется библиотека OpenSSL SSL: ${OPENSSL_SSL_LIBRARY}")
message(STATUS "Используется библиотека OpenSSL Crypto: ${OPENSSL_CRYPTO_LIBRARY}")

set(LUA_INCLUDE_DIR ${LUAJIT_INCLUDE_DIR} CACHE PATH "Путь к заголовочным файлам Lua")

set(LUASEC_SOURCES
    src/options.c
    src/config.c
    src/ec.c
    src/x509.c
    src/context.c
    src/ssl.c
    # src/luasocket/buffer.c
    # src/luasocket/io.c
    # src/luasocket/timeout.c
    # src/luasocket/usocket.c
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