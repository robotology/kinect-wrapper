# Copyright: (C) 2014 iCub Facility - Istituto Italiano di Tecnologia
# Authors: Ilaria Gori
# CopyPolicy: Released under the terms of the GNU GPL v2.0.

include(CMakeDependentOption)

set(PROJECTNAME kinectWrapper)
project(${PROJECTNAME})

if (KinectSDK_FOUND AND (NOT OpenNI_FOUND))
  message("Found KinectSDK only.")
  set(USE_KinectSDK TRUE)
elseif ((NOT KinectSDK_FOUND) AND OpenNI_FOUND)
  message("Found OpenNI only.")
  set(USE_KinectSDK FALSE)
elseif ((NOT KinectSDK_FOUND) AND (NOT OpenNI_FOUND))
  if(NOT BUILD_CLIENT_ONLY)
    message("Neither OpenNI nor KinectSDK are found. Only the client will be built.")
  endif()  
else()
  message("Found both KinectSDK and OpenNI. Choose using cmake option (sdk by default.)")
  option(USE_KinectSDK "" TRUE)
endif()

if(NOT USE_KinectSDK)
  message(STATUS "USE_KinectSDK is OFF, we will use OpenNI")
endif()

include_directories(${PROJECT_SOURCE_DIR}/include
                    ${YARP_INCLUDE_DIRS}
                    ${OpenCV_INCLUDE_DIRS}) 

set(headers_pub include/kinectWrapper/kinectTags.h
                include/kinectWrapper/kinectWrapper.h
                include/kinectWrapper/kinectWrapper_client.h)
set(sources src/kinectWrapper_client.cpp)

if (USE_KinectSDK AND KinectSDK_FOUND)
   include_directories(${KinectSDK_INCLUDE_DIRS})
   LINK_DIRECTORIES(${KinectSDK_LIB_DIR})
   add_definitions(-D__USE_SDK__)
   set(headers_priv include/kinectWrapper/kinectDriver.h
                    include/kinectWrapper/kinectDriverSDK.h
                    include/kinectWrapper/kinectWrapper_server.h)
   list(APPEND sources src/kinectDriverSDK.cpp
                       src/kinectWrapper_server.cpp)
elseif ((NOT USE_KinectSDK) AND OpenNI_FOUND)
   include_directories(${OpenNI_INCLUDE_DIRS})
   add_definitions(-D__USE_OPENNI__)
   set(headers_priv include/kinectWrapper/kinectDriver.h
                    include/kinectWrapper/kinectDriverOpenNI.h
                    include/kinectWrapper/kinectWrapper_server.h)
   list(APPEND sources src/kinectDriverOpenNI.cpp
                       src/kinectWrapper_server.cpp)
endif ()
set(headers ${headers_pub} ${headers_priv})

source_group("Header Files" FILES ${headers})
source_group("Source Files" FILES ${sources})

add_library(${PROJECTNAME} ${headers} ${sources})

if (WIN32)
    set_target_properties(${PROJECTNAME} PROPERTIES DEBUG_POSTFIX "d")
endif ()

target_link_libraries(${PROJECTNAME} ${YARP_LIBRARIES} ${OpenCV_LIBRARIES})

if (USE_KinectSDK AND KinectSDK_FOUND AND (NOT BUILD_CLIENT_ONLY))
    target_link_libraries(${PROJECTNAME} ${KinectSDK_LIBRARIES})
    icubcontrib_export_library(${PROJECTNAME} INTERNAL_INCLUDE_DIRS ${PROJECT_SOURCE_DIR}/include
                                              EXTERNAL_INCLUDE_DIRS ${YARP_INCLUDE_DIRS} ${OpenCV_INCLUDE_DIRS}
                                              DESTINATION include/kinectWrapper
                                              FILES ${headers_pub})
elseif ((NOT USE_KinectSDK) AND OpenNI_FOUND AND (NOT BUILD_CLIENT_ONLY))
    target_link_libraries(${PROJECTNAME} ${OpenNI_LIBRARIES})
    icubcontrib_export_library(${PROJECTNAME} INTERNAL_INCLUDE_DIRS ${PROJECT_SOURCE_DIR}/include
                                              EXTERNAL_INCLUDE_DIRS ${YARP_INCLUDE_DIRS} ${OpenCV_INCLUDE_DIRS}
                                              DESTINATION include/kinectWrapper
                                              FILES ${headers_pub})
else ()
    icubcontrib_export_library(${PROJECTNAME} INTERNAL_INCLUDE_DIRS ${PROJECT_SOURCE_DIR}/include
                                              EXTERNAL_INCLUDE_DIRS ${YARP_INCLUDE_DIRS} ${OpenCV_INCLUDE_DIRS}
                                              DESTINATION include/kinectWrapper
                                              FILES ${headers_pub})
endif ()



