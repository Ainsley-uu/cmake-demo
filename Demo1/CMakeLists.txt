# Cmake 指定运行此配置文件所需的CMake的最低版本
cmake_minimum_required (VERSION 2.8)
# 项目信息
# project:参数值是Demo1,该命令表示项目的名称是Demo1
project (Demo1)
# add_executable:将名为main.cc的源文件编译成一个名称为Demo的可执行文件
add_executable(Demo main.cc)
