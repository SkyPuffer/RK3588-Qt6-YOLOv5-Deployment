cmake_minimum_required(VERSION 3.16)
project(YOLOv5_RKNN_Qt)

# ⚠️ 极度关键：Qt6 强制要求必须使用 C++17 标准！
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 开启 Qt 的三大自动化魔法（不加这三行，你的界面绝对跑不起来）
set(CMAKE_AUTOMOC ON) # 自动处理信号与槽
set(CMAKE_AUTORCC ON) # 自动处理图片、图标等资源文件
set(CMAKE_AUTOUIC ON) # 自动把 .ui 文件转成 C++ 代码

# 寻找板子上的 Qt6 核心组件
find_package(Qt6 COMPONENTS Core Gui Widgets REQUIRED)

# 告诉编译器：去咱们的 include 文件夹里找 NPU 字典
include_directories(${CMAKE_SOURCE_DIR}/include)

# 告诉编译器：去咱们的 lib 文件夹里找 NPU 翻译官
link_directories(${CMAKE_SOURCE_DIR}/lib)

# 编译咱们的 main.cpp
add_executable(yolov5_app src/main.cpp)

# 终极绑定：把 Qt6 的库 和 NPU 的库(rknnrt) 全部链接到咱们的程序上
target_link_libraries(yolov5_app
    Qt6::Core
    Qt6::Gui
    Qt6::Widgets
    rknnrt
)