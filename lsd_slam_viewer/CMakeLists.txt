cmake_minimum_required(VERSION 2.8.3)
project(lsd_slam_viewer)

add_subdirectory(${PROJECT_SOURCE_DIR}/thirdparty/Sophus)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/cmake)

find_package(catkin REQUIRED
    COMPONENTS
        message_generation
        std_msgs
        dynamic_reconfigure)

find_package(OpenGL REQUIRED)
set(QT_USE_QTOPENGL TRUE)
set(QT_USE_QTXML TRUE)

find_package(QGLViewer REQUIRED)
find_package(OpenCV REQUIRED)
find_package(Eigen REQUIRED)
find_package(Boost REQUIRED COMPONENTS thread)

include_directories(
    include
    ${catkin_INCLUDE_DIRS}
    ${PROJECT_SOURCE_DIR}/thirdparty/Sophus
    ${QT_INCLUDES}
    ${OpenCV_INCLUDE_DIRS}
    ${EIGEN_INCLUDE_DIRS}
    ${QGLVIEWER_INCLUDE_DIR}
)

generate_dynamic_reconfigure_options(cfg/LSDSLAMViewerParams.cfg)

add_message_files(
    DIRECTORY msg
    FILES
        keyframeGraphMsg.msg
        keyframeMsg.msg)

generate_messages(DEPENDENCIES std_msgs)

# SSE flags
# @todo LOOK HOW TO DO THIS WITH PLAIN CMAKE
#rosbuild_check_for_sse()
#set(CMAKE_CXX_FLAGS "${SSE_FLAGS}")

catkin_package(
    LIBRARIES ${PROJECT_NAME}
    CATKIN_DEPENDS
        message_runtime
        std_msgs
        dynamic_reconfigure)

add_library(${PROJECT_NAME}
    src/PointCloudViewer.cpp
    src/KeyFrameDisplay.cpp
    src/KeyFrameGraphDisplay.cpp
    src/settings.cpp)
add_dependencies(${PROJECT_NAME} ${PROJECT_NAME}_gencfg ${PROJECT_NAME}_generate_messages_cpp)
target_link_libraries(${PROJECT_NAME} ${catkin_LIBRARIES} ${QGLViewer_LIBRARIES} ${QT_LIBRARIES} ${QGLVIEWER_LIBRARY} ${OpenCV_LIBRARIES} ${Boost_LIBRARIES} GL glut GLU)

add_executable(viewer src/main_viewer.cpp ${SOURCE_FILES} ${HEADER_FILES})
target_link_libraries(viewer ${PROJECT_NAME})

add_executable(videoStitch src/main_stitchVideos.cpp)
target_link_libraries(videoStitch ${PROJECT_NAME})

install(TARGETS viewer videoStitch
    ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
    RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(DIRECTORY include/${PROJECT_NAME}
    DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
    FILES_MATCHING PATTERN "*.h")
