cmake_minimum_required(VERSION 3.22...4.0)
project(vulkandraw NONE)

# Cesta ke Go souboru
set(GO_SOURCE ${CMAKE_CURRENT_SOURCE_DIR}/app_android/main.go)
set(SO_OUTPUT ${CMAKE_LIBRARY_OUTPUT_DIRECTORY}/libvulkandraw.so)

set(ANDROID_NDK_HOME ${CMAKE_ANDROID_NDK})
set(ANDROID_TOOLCHAIN_DIR ${CMAKE_ANDROID_NDK}/toolchains/llvm/prebuilt/linux-x86_64/bin)
set(ANDROID_API 25)

set(CC ${ANDROID_TOOLCHAIN_DIR}/aarch64-linux-android${ANDROID_API}-clang)
set(CXX ${ANDROID_TOOLCHAIN_DIR}/aarch64-linux-android${ANDROID_API}-clang++)
set(GOOS android)
set(GOARCH arm64)

# IMPORTED allows dependency on a library file outside the project.
add_library(go-lib SHARED IMPORTED)

# IMPORTED_LOCATION specifies the location of the library file on disk
set_target_properties(go-lib PROPERTIES IMPORTED_LOCATION ${SO_OUTPUT})

# Add the library to this project
target_link_libraries(
        ${PROJECT_NAME}
        go-lib
)

# Cíl pro sestavení knihovny
add_custom_target(build-go-lib ALL DEPENDS ${SO_OUTPUT})

# Příkaz pro sestavení .so knihovny pomocí Go
add_custom_command(
        OUTPUT ${SO_OUTPUT}
#        TARGET build_go_lib
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        COMMAND CGO_ENABLED=1 CC=${CC} CXX=${CXX} GOOS=${GOOS} GOARCH=${GOARCH} go build -buildmode=c-shared -o ${SO_OUTPUT} ${GO_SOURCE}
        VERBATIM
#        BYPRODUCTS ${SO_OUTPUT}
        COMMENT "Building Go shared library"
)

add_dependencies(go-lib build-go-lib)
