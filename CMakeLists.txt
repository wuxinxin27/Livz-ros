cmake_minimum_required(VERSION 3.0.2)
project(livz)

# 设置C++标准为C++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -fPIC")

# 添加ROS include目录
find_package(catkin REQUIRED COMPONENTS
  roscpp
  visualization_msgs
  geometry_msgs
  sensor_msgs
)

# 寻找PCL库
find_package(PCL REQUIRED)


# 寻找Eigen库
find_package(Eigen3 REQUIRED)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES ${PROJECT_NAME}
  CATKIN_DEPENDS roscpp visualization_msgs geometry_msgs sensor_msgs
  DEPENDS PCL EIGEN3
)

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${PCL_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}
)

add_definitions(-DINSTALL_PREFIX="${CMAKE_INSTALL_PREFIX}")
add_definitions(${PCL_DEFINITIONS}) # 添加PCL定义

execute_process(COMMAND bash -c "echo $HOME" OUTPUT_VARIABLE HOME_DIR OUTPUT_STRIP_TRAILING_WHITESPACE)
execute_process(
    COMMAND bash -c
    "if grep -q 'export LIVZ_TOUCH_LIB_VAR=' ${HOME_DIR}/.bashrc; then \
        sed -i 's|export LIVZ_TOUCH_LIB_VAR=.*|export LIVZ_TOUCH_LIB_VAR=${CMAKE_INSTALL_PREFIX}/lib/LiTouch/scripts|' ${HOME_DIR}/.bashrc; \
    else \
        echo 'export LIVZ_TOUCH_LIB_VAR=${CMAKE_INSTALL_PREFIX}/lib/LiTouch/scripts' >> ${HOME_DIR}/.bashrc; \
    fi"
)

execute_process(
    COMMAND bash -c
    "if [ -f ${HOME_DIR}/.zshrc ]; then \
        if grep -q 'export LIVZ_TOUCH_LIB_VAR=' ${HOME_DIR}/.zshrc; then \
            sed -i 's|export LIVZ_TOUCH_LIB_VAR=.*|export LIVZ_TOUCH_LIB_VAR=${CMAKE_INSTALL_PREFIX}/lib/LiTouch/scripts|' ${HOME_DIR}/.zshrc; \
        else \
            echo 'export LIVZ_TOUCH_LIB_VAR=${CMAKE_INSTALL_PREFIX}/lib/LiTouch/scripts' >> ${HOME_DIR}/.zshrc; \
        fi \
    fi"
)


set(CMAKE_POSITION_INDEPENDENT_CODE ON)
add_library(${PROJECT_NAME} STATIC
                src/livz.cpp
                src/livz_animate.cpp)

# 添加ROS链接库
target_link_libraries( ${PROJECT_NAME} PUBLIC
    ${catkin_LIBRARIES}
    ${PCL_LIBRARIES}
)


# 安装静态链接库
install(TARGETS ${PROJECT_NAME}
        ARCHIVE DESTINATION lib
        LIBRARY DESTINATION lib)

# 安装头文件
install(DIRECTORY include/
        DESTINATION include/livz)

# 安装LivzTouch
install(DIRECTORY LiTouch/
DESTINATION lib/LiTouch)

install(FILES LiTouch/config/default_action.py
  DESTINATION lib/LiTouch/config
  PERMISSIONS OWNER_WRITE OWNER_READ GROUP_WRITE GROUP_READ WORLD_WRITE WORLD_READ
)