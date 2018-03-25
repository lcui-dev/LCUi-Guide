# 在 Linux 系统中安装

这里假设你已经拥有完整的开发环境，包括构建此项目所需的相关工具（例如：gcc、automake），如果你在按照本章节的方法构建本项目时发现缺少某些工具，请自行安装它们。

## 安装依赖项

如果你的系统是 Ubuntu，可运行以下命令来安装依赖：

    apt-get install libpng-dev libjpeg-dev libxml2-dev libfreetype6-dev libx11-dev

## 安装 LCUI

在最简单的情况下，你可以运行以下命令行：

    git clone https://github.com/lc-soft/LCUI.git
    cd LCUI
    ./autogen.sh
    ./configure

在 configure 脚本执行完后，运行以下命令编译源代码并安装 LCUI 的函数库和头文件：

    make
    make install

如果需要运行示例程序，可运行命令来编译生成示例程序：

    cd test
    make

## 配置

为编译器添加参数，指定 LCUI 的头文件和库文件的查找位置，例如：

    gcc -c test.c `pkg-config --cflags lcui`
    gcc –o test.o test `pkg-config --libs lcui`

通常情况下，你也可以直接用以下方法：

    gcc -c test.c
    gcc –o test.o test -lLCUI

## 完成

至此，你已经完成了 LCUI 开发环境的搭建，接下来你可以开始尝试[编写 Hello World](../getting_started/step1.html)。
