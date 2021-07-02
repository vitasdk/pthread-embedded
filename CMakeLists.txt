cmake_minimum_required(VERSION 3.0)

project(pthread-embedded LANGUAGES C)

set(CMAKE_C_STANDARD 99)

option(BUILD_TESTSUITE "Build testsuite" OFF)

if (VITA)
  include("$ENV{VITASDK}/share/vita.cmake" REQUIRED)
endif()

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wl,-q -Wall -O3 -fno-strict-aliasing")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -fno-rtti -fno-exceptions -Werror -D__CLEANUP_CXX")

include_directories(${CMAKE_SOURCE_DIR})

if(VITA)
  include_directories(${CMAKE_SOURCE_DIR}/platform/vita/)
  file(GLOB PLATFORM_SOURCES ${CMAKE_SOURCE_DIR}/platform/vita/vita_osal.c)

  set(VITA_APP_NAME "PTHREAD TEST")
  set(VITA_TITLEID  "PTRD00000")
  set(VITA_VERSION  "01.00")
  set(VITA_MKSFOEX_FLAGS "${VITA_MKSFOEX_FLAGS} -d PARENTAL_LEVEL=1")
elseif (PLATFORM_PSP)
  include_directories(${CMAKE_SOURCE_DIR}/platform/psp/)
  include_directories(${CMAKE_SOURCE_DIR}/platform/helper/)
  link_directories(${PSPSDK}/lib ${PSPDEV}/lib)
  file(GLOB PLATFORM_SOURCES ${CMAKE_SOURCE_DIR}/platform/psp/psp_osal.c)
  set(PSP_APP_NAME "PTHREAD TEST")
endif()

file(GLOB SOURCES ${CMAKE_SOURCE_DIR}/*.c)
file(GLOB TEST_SOURCES ${CMAKE_SOURCE_DIR}/tests/*.c)
file(GLOB HELPER_SOURCES ${CMAKE_SOURCE_DIR}/platform/helper/*.c)

if (VITA)
    list(APPEND TEST_SOURCES ${CMAKE_SOURCE_DIR}/platform/vita/main.c)
elseif (PLATFORM_PSP)
    list(APPEND PLATFORM_SOURCES ${HELPER_SOURCES})
    list(APPEND TEST_SOURCES ${CMAKE_SOURCE_DIR}/platform/psp/main.c)
endif()

add_library(pthread STATIC
  ${SOURCES}
  ${PLATFORM_SOURCES}
)

if (BUILD_TESTSUITE)

  add_executable(pthread-test
    ${TEST_SOURCES}
  )

  target_link_libraries(pthread-test pthread)

  if(VITA)
    vita_create_self(pthread-test.self pthread-test UNSAFE)

    vita_create_vpk(pthread-test.vpk ${VITA_TITLEID} pthread-test.self
      VERSION ${VITA_VERSION}
      NAME ${VITA_APP_NAME}
      FILE platform/vita/sce_sys sce_sys
    )
  elseif (PLATFORM_PSP)
    target_link_libraries(pthread-test
      ${PSP_LIBRARIES}
    )
    create_pbp_file(
      TARGET pthread-test
      TITLE ${PSP_APP_NAME}
    )
  endif()
endif()

install(FILES
    pthread_public.h
    DESTINATION ${CMAKE_INSTALL_PREFIX}/include/
    RENAME pthread.h
)

install(FILES
    sched.h semaphore.h
    DESTINATION ${CMAKE_INSTALL_PREFIX}/include/
)

if(VITA)
  install(FILES
    platform/vita/pte_types.h
    DESTINATION ${CMAKE_INSTALL_PREFIX}/include/
  )
elseif (PLATFORM_PSP)
  install(FILES
    platform/psp/pte_types.h
    DESTINATION ${CMAKE_INSTALL_PREFIX}/include/
  )
endif()

install(TARGETS pthread ARCHIVE)

