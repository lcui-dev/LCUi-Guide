# 安装

这里假设你已经拥有完整的开发环境，包括构建此项目所需的相关工具，如果你在按照本章节的方法构建本项目时发现缺少某些工具，请自行安装它们。

## 引导

你需要运行源码根目录中的 configure 脚本文件以引导项目的构建。

在最简单的情况下，你可以运行：

	git clone https://github.com/lc-soft/LCUI.git
	cd LCUI
	./configure

如果未找到 configure，请运行 autogen.sh 脚本生成它。

在 configure 脚本执行完后，运行以下命令编译源代码并安装 LCUI 的函数库和头文件：

	make
	make install

如果需要运行示例程序，可运行命令来编译生成示例程序：

	cd test
	make

## 依赖项

如果你想构建全特性的 LCUI，建议安装以下依赖库：

 * [libpng](http://www.libpng.org/pub/png/libpng.html) — PNG 图像压缩库
 * [libjpeg](http://www.ijg.org/) — JPEG 图像压缩库
 * [libxml2](http://xmlsoft.org/) — XML 解析器及相关工具集
 * [libx11](https://www.x.org/) — X11 客户端库
 * [freetype](https://www.freetype.org/) — 字体渲染引擎

如果你的系统是 Ubuntu，可运行以下命令来安装依赖：

	apt-get install libpng-dev libjpeg-dev libxml2-dev libfreetype6-dev libx11-dev


## 在 Windows 中构建

LCUI 主要是在 Windows 系统环境下开发的，你可以使用 VisualStudio 打开 
/build/VS2012/LCUI.sln 文件，然后在 VisualStudio 的菜单中选择 生成 -> 生成解决方案 来编译生成 LCUI。如果你用的是其它 IDE，请尝
试按该 IDE 的方式创建项目并将源文件添加至项目内，然后编译。

上述的依赖库除 windows 用不到的 libx11 库外，都可以在 Windows 系统环境下编译生成，如果觉得手动编译它们很
麻烦，想要现成可用的依赖库和头文件，可以在网上搜索，或者联系作者。
