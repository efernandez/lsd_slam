cmake_minimum_required(VERSION 2.8.3)
project(lsd_slam_core)

set(CMAKE_MODULE_PATH   ${PROJECT_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})

find_package(catkin REQUIRED
    COMPONENTS
        sensor_msgs
        dynamic_reconfigure
        cv_bridge
        lsd_slam_viewer)

find_package(Eigen REQUIRED)
find_package(SuiteParse REQUIRED)
find_package(Boost REQUIRED COMPONENTS thread)

# FabMap
# uncomment this part to enable fabmap
#add_subdirectory(${PROJECT_SOURCE_DIR}/thirdparty/openFabMap)
#include_directories(${PROJECT_SOURCE_DIR}/thirdparty/openFabMap/include)
#add_definitions("-DHAVE_FABMAP")
#set(FABMAP_LIB openFABMAP )

generate_dynamic_reconfigure_options(
    cfg/LSDDebugParams.cfg
    cfg/LSDParams.cfg)

# SSE flags
# @todo LOOK HOW TO DO THIS WITH PLAIN CMAKE
#rosbuild_check_for_sse()
add_definitions("-DUSE_ROS")
#add_definitions("-DENABLE_SSE")

# Also add some useful compiler flag
set(CMAKE_CXX_FLAGS
   "${CMAKE_CXX_FLAGS} ${SSE_FLAGS} -march=native -std=c++0x"
)

# Set source files
set(lsd_SOURCE_FILES
    ${PROJECT_SOURCE_DIR}/src/DataStructures/Frame.cpp
    ${PROJECT_SOURCE_DIR}/src/DataStructures/FramePoseStruct.cpp
    ${PROJECT_SOURCE_DIR}/src/DataStructures/FrameMemory.cpp
    ${PROJECT_SOURCE_DIR}/src/SlamSystem.cpp
    ${PROJECT_SOURCE_DIR}/src/LiveSLAMWrapper.cpp
    ${PROJECT_SOURCE_DIR}/src/DepthEstimation/DepthMap.cpp
    ${PROJECT_SOURCE_DIR}/src/DepthEstimation/DepthMapPixelHypothesis.cpp
    ${PROJECT_SOURCE_DIR}/src/util/globalFuncs.cpp
    ${PROJECT_SOURCE_DIR}/src/util/SophusUtil.cpp
    ${PROJECT_SOURCE_DIR}/src/util/settings.cpp
    ${PROJECT_SOURCE_DIR}/src/util/Undistorter.cpp
    ${PROJECT_SOURCE_DIR}/src/Tracking/Sim3Tracker.cpp
    ${PROJECT_SOURCE_DIR}/src/Tracking/Relocalizer.cpp
    ${PROJECT_SOURCE_DIR}/src/Tracking/SE3Tracker.cpp
    ${PROJECT_SOURCE_DIR}/src/Tracking/least_squares.cpp
    ${PROJECT_SOURCE_DIR}/src/Tracking/TrackingReference.cpp
    ${PROJECT_SOURCE_DIR}/src/IOWrapper/Timestamp.cpp
    ${PROJECT_SOURCE_DIR}/src/GlobalMapping/FabMap.cpp
    ${PROJECT_SOURCE_DIR}/src/GlobalMapping/KeyFrameGraph.cpp
    ${PROJECT_SOURCE_DIR}/src/GlobalMapping/g2oTypeSim3Sophus.cpp
    ${PROJECT_SOURCE_DIR}/src/GlobalMapping/TrackableKeyFrameSearch.cpp
)
set(SOURCE_FILES
    ${lsd_SOURCE_FILES}
    ${PROJECT_SOURCE_DIR}/src/IOWrapper/ROS/ROSImageStreamThread.cpp
    ${PROJECT_SOURCE_DIR}/src/IOWrapper/ROS/ROSOutput3DWrapper.cpp
    ${PROJECT_SOURCE_DIR}/src/IOWrapper/OpenCV/ImageDisplay_OpenCV.cpp
)

include_directories(
    include
    ${catkin_INCLUDE_DIRS}
    ${PROJECT_SOURCE_DIR}/src
    ${PROJECT_SOURCE_DIR}/thirdparty/Sophus
    ${EIGEN_INCLUDE_DIR}
    ${CSPARSE_INCLUDE_DIR} # set by SuiteParse
    ${CHOLMOD_INCLUDE_DIR} # set by SuiteParse
)

catkin_package(
    LIBRARIES ${PROJECT_NAME}
    CATKIN_DEPENDS
        sensor_msgs
        dynamic_reconfigure
        cv_bridge
        lsd_slam_viewer)

# build shared library.
add_library(${PROJECT_NAME} SHARED ${SOURCE_FILES})
add_dependencies(${PROJECT_NAME} ${PROJECT_NAME}_gencfg)
target_link_libraries(${PROJECT_NAME} ${FABMAP_LIB} ${Boost_LIBRARIES} g2o_core g2o_stuff csparse cxsparse g2o_solver_csparse g2o_csparse_extension g2o_types_sim3 g2o_types_sba)

# build live ros node
add_executable(live_slam src/main_live_odometry.cpp)
target_link_libraries(live_slam lsdslam)

# build image node
add_executable(dataset src/main_on_images.cpp)
target_link_libraries(dataset lsdslam)

