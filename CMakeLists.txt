cmake_minimum_required(VERSION 2.8)

if(NOT DEFINED CMAKE_TOOLCHAIN_FILE)
  if(DEFINED ENV{VITASDK})
    set(CMAKE_TOOLCHAIN_FILE "$ENV{VITASDK}/share/vita.toolchain.cmake" CACHE PATH "toolchain file")
  else()
    message(FATAL_ERROR "Please define VITASDK to point to your SDK path!")
  endif()
endif()

set(CMAKE_SYSTEM_NAME "Generic")
set(CMAKE_C_COMPILER "arm-vita-eabi-gcc")
set(CMAKE_CXX_COMPILER "arm-vita-eabi-g++")

project(ds4vita)
include("${VITASDK}/share/vita.cmake" REQUIRED)

set(CMAKE_C_FLAGS "-Wl,-q -Wall -O3 -std=gnu99")
set(CMAKE_CXX_FLAGS "${CMAKE_C_FLAGS} -std=c++11 -fno-rtti -fno-exceptions")

include_directories(
)

link_directories(
	${CMAKE_CURRENT_BINARY_DIR}
)

option(RELEASE "Release build" OFF)

if (RELEASE)
	add_definitions(-DRELEASE)
endif(RELEASE)

add_executable(${PROJECT_NAME}
	main.c
	log.c
)

target_link_libraries(${PROJECT_NAME}
	gcc
	taihenForKernel_stub
	SceSysclibForDriver_stub
	SceSysmemForDriver_stub
	SceSysmemForKernel_stub
	SceThreadmgrForDriver_stub
	SceIofilemgrForDriver_stub
	SceBtForDriver_stub
	SceKernelSuspendForDriver_stub
	SceCtrlForDriver_stub
)

set_target_properties(${PROJECT_NAME}
	PROPERTIES LINK_FLAGS "-nostdlib"
	COMPILE_FLAGS "-D__VITA_KERNEL__"
)

vita_create_self(${PROJECT_NAME}.skprx ${PROJECT_NAME}
  UNSAFE
  CONFIG ${PROJECT_NAME}.yml
)

install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/stubs/
  DESTINATION lib
  FILES_MATCHING PATTERN "*.a"
)
