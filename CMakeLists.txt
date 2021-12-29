# 版本信息
CMAKE_MINIMUM_REQUIRED(VERSION 3.1)

# out-of-source编译
if(${CMAKE_SOURCE_DIR} STREQUAL ${CMAKE_BINARY_DIR})
  message(FATAL_ERROR "Prevented in-tree built. Please create a build directory outside of the source code and call cmake from there")
endif()

# 工程名称，变量${PROJECT_NAME}
project(HelloCMake)

################################
# Android
# cmake -DCMAKE_SYSTEM_NAME=Android -DANDROID_PLATFORM=23 -DCMAKE_ANDROID_ARCH_ABI=armeabi-v7a -DCMAKE_TOOLCHAIN_FILE=${ANDROID_NDK}/build/cmake/android.toolchain.cmake 
################################
# set(CMAKE_SYSTEM_NAME Android)
# set(CMAKE_SYSTEM_VERSION 23) # API level
# set(CMAKE_TOOLCHAIN_FILE $ENV{NDROID_NDK}/build/cmake/android.toolchain.cmake)
# set(CMAKE_ANDROID_ARCH_ABI armeabi-v7a)
# set(CMAKE_ANDROID_STL_TYPE c++_static)
# set(CMAKE_ANDROID_NDK $ENV{NDROID_NDK})
# set(CMAKE_ANDROID_ARM_MODE arm)
# set(ANDROID_PLATFORM android-23)

set(CMAKE_BUILD_TYPE Release)

set(CMAKE_STRIP "${CMAKE_STRIP} -x")

################################
# 头文件检查
################################
# 包含CHECK_INCLUDE_FILE_CXX函数
include(CheckIncludeFileCXX)
# algorithm头文件
CHECK_INCLUDE_FILE_CXX(algorithm HAS_ALGORITHM)
message(HAS_ALGORITHM=${HAS_ALGORITHM})

################################
# 工程目录
# CMAKE_CURRENT_SOURCE_DIR - 当前CMakeLists.txt的目录
# CMAKE_CURRRENT_BINARY_DIR - 当前target编译目录；in source，和CMAKE_CURRENT_SOURCE_DIR一致；out-of-source，当前target编译目录
# CMAKE_SOURCE_DIR/PROJECT_SOURCE_DIR/<projectname>_SOURCE_DIR(PROJECT_SOURCE_DIR) - 顶层源码目录
# CMAKE_BINARY_DIR/PROJECT_BINARY_DIR/<projectname>_BINARY_DIR(PROJECT_BINARY_DIR) - target编译目录，in source编译: 指得就是工程顶层目录; out-of-source编译: 指的是工程编译发生的目录
# SET(EXECUTABLE_OUTPUT_PATH <新路径>)并不会对这个变量造成影响,它仅仅修改了最终目标文件存放的路径
################################
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
set(CMAKE_INSTALL_PREFIX ${PROJECT_BINARY_DIR}/export)

################################
# 子目录
################################
# add_subdirectory(source_dir [binary_dir] [EXCLUDE_FROM_ALL])

################################
# ASM开关
################################
enable_language(ASM)
set(can_use_assembler TRUE)

################################
# 使用相对路径
################################
set(CMAKE_SUPPRESS_REGENERATION 1)
set(CMAKE_USE_RELATIVE_PATHS ON)

################################
# 编译选项
# -fno-omit-frame-pointer
# -fsanitize=address
# -fvisibility=hidden
# -fshort-wchar
# -std=c++11
# -g
# -fno-exceptions
# -Werror=return-type
# -Wno-unused-parameter
# -Wno-reorder
# -Wno-unnamed-type-template-args
################################
set(CMAKE_C_FLAGS,  "${CMAKE_C_FLAGS}   -fsigned-char -O3 -fvisibility=hidden ")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsigned-char -std=c++11 -O3 -fvisibility=hidden -Wall ")

# 宏定义，或在编译选项中添加
add_definitions(-DNDEBUG )

# 头文件路径，或编译选项-I
include_directories(dir1 dir2 dir3)

# 库文件路径，或编译选项-L
link_directories(dir1 dir2 dir3)

# 链接的库列表
link_libraries(lib1 lib2)

# 链接选项
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -Wl")

################################
# 依赖关系
################################
add_dependencies(APP GUI)

################################
# 源文件
################################
set(SRC_FILES main.cpp)
# list(APPEND SRC_FILES d.c e.cpp)

################################
# 源文件分组
# source_group(name [REGULAR_EXPRESSION regex] [FILES src1 src2 ...])
################################
macro(source_group_by_dir source_files)
    if(MSVC)
        set(sgbd_cur_dir ${CMAKE_CURRENT_SOURCE_DIR})
        foreach(sgbd_file ${${source_files}})
            string(REGEX REPLACE ${sgbd_cur_dir}/\(.*\) \\1 sgbd_fpath ${sgbd_file})
            string(REGEX REPLACE "\(.*\)/.*" \\1 sgbd_group_name ${sgbd_fpath})
            string(COMPARE EQUAL ${sgbd_fpath} ${sgbd_group_name} sgbd_nogroup)
            string(REPLACE "/" "\\" sgbd_group_name ${sgbd_group_name})
            if(sgbd_nogroup)
                set(sgbd_group_name "\\")
            endif(sgbd_nogroup)
            source_group(${sgbd_group_name} FILES ${sgbd_file})
        endforeach(sgbd_file)
    endif(MSVC)
endmacro(source_group_by_dir)

source_group("Hello\\A" FILES ${SRC_FILES})
# source_group_by_dir(SRC_FILES)

################################
# 编译目标 SHARED|STATIC
################################
set(TargetName ${PROJECT_NAME})
# add_library(${TargetName} SHARED ${SRC_FILES})
add_executable(${TargetName} ${SRC_FILES})

################################
# 设置目标选项
################################
target_compile_options(${TargetName} PRIVATE $<$<COMPILE_LANGUAGE:C>: -O3 -ggdb>)
target_compile_options(${TargetName} PRIVATE $<$<COMPILE_LANGUAGE:CXX>: -std=c++11 -O3 -ggdb>)
target_compile_definitions(${TargetName} PRIVATE  GUTILITY_DLL)
target_include_directories(${TargetName} PRIVATE "dir1" "dir2")
target_precompile_headers(${TargetName} PRIVATE "stdafx.h")

target_link_libraries(${TargetName} lib1 lib2)
# -Wl,--no-wchar-size-warning
set_target_properties(${TargetName} PROPERTIES LINK_FLAGS " ")

################################
# install
################################
install(TARGETS ${TargetName} DESTINATION bin)
