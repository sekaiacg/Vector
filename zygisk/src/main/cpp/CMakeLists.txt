cmake_minimum_required(VERSION 3.10)

project(zygisk)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(${VECTOR_ROOT}/native native)

add_library(${PROJECT_NAME} SHARED module.cpp ipc_bridge.cpp hyos_runtime.cpp)
target_include_directories(${PROJECT_NAME} PUBLIC include)

target_link_libraries(${PROJECT_NAME} native log)

set(DEX2OAT_INJECT dex2oat)
add_library(${DEX2OAT_INJECT} SHARED dex2oat_inject.cpp)
target_include_directories(${DEX2OAT_INJECT} PUBLIC include)
target_link_libraries(${DEX2OAT_INJECT} native log)

if (DEFINED DEBUG_SYMBOLS_PATH)
    message(STATUS "Debug symbols will be placed at ${DEBUG_SYMBOLS_PATH}")
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E make_directory ${DEBUG_SYMBOLS_PATH}/${ANDROID_ABI}
            COMMAND ${CMAKE_OBJCOPY} --only-keep-debug $<TARGET_FILE:${PROJECT_NAME}>
            ${DEBUG_SYMBOLS_PATH}/${ANDROID_ABI}/${PROJECT_NAME}.debug
            COMMAND ${CMAKE_STRIP} --strip-all $<TARGET_FILE:${PROJECT_NAME}>
            COMMAND ${CMAKE_OBJCOPY} --add-gnu-debuglink ${DEBUG_SYMBOLS_PATH}/${ANDROID_ABI}/${PROJECT_NAME}.debug
            $<TARGET_FILE:${PROJECT_NAME}>)
    add_custom_command(TARGET ${DEX2OAT_INJECT} POST_BUILD
            COMMAND ${CMAKE_COMMAND} -E make_directory ${DEBUG_SYMBOLS_PATH}/${ANDROID_ABI}
            COMMAND ${CMAKE_OBJCOPY} --only-keep-debug $<TARGET_FILE:${DEX2OAT_INJECT}>
            ${DEBUG_SYMBOLS_PATH}/${ANDROID_ABI}/${DEX2OAT_INJECT}.debug
            COMMAND ${CMAKE_STRIP} --strip-all $<TARGET_FILE:${DEX2OAT_INJECT}>
            COMMAND ${CMAKE_OBJCOPY} --add-gnu-debuglink ${DEBUG_SYMBOLS_PATH}/${ANDROID_ABI}/${DEX2OAT_INJECT}.debug
            $<TARGET_FILE:${DEX2OAT_INJECT}>)
endif()
