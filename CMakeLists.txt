#Copyright (c) Microsoft. All rights reserved.
#Licensed under the MIT license. See LICENSE file in the project root for full license information.

cmake_minimum_required(VERSION 2.8.12)

# Check if the environment variable JAVA_HOME exists
if(
    (NOT (DEFINED ENV{JAVA_HOME}))
  )
  message(FATAL_ERROR "Environment variable JAVA_HOME is not defined. Please define "
                      "JAVA_HOME to point to the location of the JDK.")
endif()

# Check if the local or environment variable JDK_ARCH exists and if not set it to 'amd64' by default
if(
    (NOT (DEFINED JDK_ARCH))
  )
  if( DEFINED ENV{JDK_ARCH} )
    set( JDK_ARCH $ENV{JDK_ARCH} )
  else()
    set( JDK_ARCH amd64 )
  endif()
endif()

set(jnano_sources
    ./src/main/c/NanoLibrary.c
)

set(jnano_headers
    ./src/main/c/NanoUtil.h
    ./src/main/c/org_nanomsg_NanoLibrary.h
)

if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  set(LINUX TRUE)
endif()

if(WIN32)
    set(java_include_dirs
        $ENV{JAVA_HOME}/include 
        $ENV{JAVA_HOME}/include/win32
    )
    set(java_link_dirs
        $ENV{JAVA_HOME}/lib
        $ENV{JAVA_HOME}/jre/bin/server
    )
    set(java_libs
        $ENV{JAVA_HOME}/lib/jvm.lib
    )
elseif(LINUX)
    set(java_include_dirs
        $ENV{JAVA_HOME}/include 
        $ENV{JAVA_HOME}/include/linux
    )
    set(java_link_dirs
        $ENV{JAVA_HOME}/jre/lib/${JDK_ARCH}/server
    )
    set(java_libs
        $ENV{JAVA_HOME}/jre/lib/${JDK_ARCH}/server/libjvm.so
    )
endif()

if(
    (NOT (DEFINED dependency_install_prefix))
  )
  set(dependency_install_prefix ../../install-deps)
endif()

set(nanomsg_include ${NANOMSG_INCLUDEDIR})
if(
    (NOT (DEFINED nanomsg_include))
  )
  set(nanomsg_include ${dependency_install_prefix}/include)
endif()

set(nanomsg_lib ${NANOMSG_LIBRARY_DIRS})
if(
    (NOT (DEFINED nanomsg_lib))
  )
  set(nanomsg_lib ${dependency_install_prefix}/lib)
endif()

include_directories(${nanomsg_include}/nanomsg)   
link_directories(${nanomsg_lib})

set(LIBS ${java_libs})

include_directories(${java_include_dirs})
link_directories(${java_link_dirs})

#this builds the jnano_sources dynamic library
add_library(jnano MODULE ${jnano_headers} ${jnano_sources})
target_link_libraries(jnano ${LIBS} nanomsg)

#this build the jnano_sources static library
add_library(jnano_static STATIC ${jnano_headers} ${jnano_sources})
target_compile_definitions(jnano_static PRIVATE BUILD_MODULE_TYPE_STATIC)
target_link_libraries(jnano_static ${LIBS} nanomsg)

install(TARGETS jnano LIBRARY DESTINATION "${LIB_INSTALL_DIR}/modules")