# 在 Windows 系统中安装

通常，LCUI 每次版本更新时都会提供编译好的 LCUI 库及依赖库，你可以直接下载使用。如果你出于某些原因（例如：编译64位版本）需要手动编译一次 LCUI，那么请继续阅读以下内容。

## 编译依赖项

在 Windows 上手动编译安装依赖库比较麻烦，不过，[vcpkg](https://github.com/Microsoft/vcpkg) 可以解决这种问题，你可以尝试使用它安装这些依赖项。

以下列出了大致的编译方法，源码包可以在它们的官网中下载到，有的项目会提供 VisualStudio 工程文件，你可以直接打开它然后编译生成它，在编译完后需要将它们提供的头文件复制到编译器能找到的目录。

### libpng

- 项目主页：http://www.libpng.org/pub/png/libpng.html
- 工程文件：projects/vstudio/vstudio.sln
- 头文件：png.h、pngconfig.h、pnglibconf.h

### freetype

- 项目主页：https://www.freetype.org/
- 工程文件：builds/windows/vc2010/freetype.sln
- 头文件：include/ft2build.h、include/freetype

### jpeg

- 项目主页：http://www.ijg.org/
- 头文件：jconfig.h、jmorecfg.h、jerror.h、jpeglib.h

jpeg 库没有提供现成的 sln 工程文件，你需要按照以下步骤手动生成它。

* 打开开始菜单，找到 Visual Studio 的目录
* 打开目录里的 VisualStudio 提供的开发人员命令提示工具
* 在出现的命令窗口中使用 cd 命令切换目录到 jpeg 源码目录
* 运行命令：`nmake -f makefile.vc setup-v10`
* 关闭命令窗口
* 在源码目录中可看到生成的 jpeg.sln 文件

如果在运行 nmake 命令后出现提示 `未找到文件“win32.mak”`，请复制 `C:\Program Files (x86)\Microsoft SDKs\Windows\v7.1A\Include` 目录下的 win32.mak 文件到 jpeg 源码目录里，然后重试。

### libxml2

- 项目主页：http://xmlsoft.org/
- 工程文件：win32/VC10/libxml2.sln
- 头文件：include/libxml

如果你想对 libxml2 进行裁剪，你可以在源码目录里的 win32 目录中运行以下命令：

    cscript configure.js 

可以看到如下输出：

``` text
Microsoft (R) Windows Script Host Version 5.812
版权所有(C) Microsoft Corporation。保留所有权利。

libxml2 version: 2.9.4
Created Makefile.
Created config.h.

XML processor configuration
---------------------------
              Trio: no
     Thread safety: native
        FTP client: yes
       HTTP client: yes
    HTML processor: yes
      C14N support: yes
   Catalog support: yes
   DocBook support: yes
     XPath support: yes
  XPointer support: yes
  XInclude support: yes
     iconv support: yes
     icu   support: no
  iso8859x support: no
      zlib support: no
      lzma support: no
  Debugging module: yes
  Memory debugging: no
 Runtime debugging: no
    Regexp support: yes
    Module support: yes
      Tree support: yes
    Reader support: yes
    Writer support: yes
    Walker support: yes
   Pattern support: yes
      Push support: yes
Validation support: yes
      SAX1 support: yes
    Legacy support: yes
    Output support: yes
XML Schema support: yes
Schematron support: yes
   Python bindings: no

Win32 build configuration
-------------------------
          Compiler: msvc
  C-Runtime option: /MD
    Embed Manifest: no
     Debug symbols: no
    Static xmllint: no
    Install prefix: .
      Put tools in: $(PREFIX)\bin
    Put headers in: $(PREFIX)\include
Put static libs in: $(PREFIX)\lib
Put shared libs in: $(PREFIX)\bin
      Include path: .
          Lib path: .
```

其中 FTP 客户端、HTTP 客户端、HTML 处理器等模块是用不到的，可以使用以下命令禁用掉它们：

    cscript configure.js ftp=no http=no html=no legacy=no iconv=no catalog=no docb=no

该脚本会更新 config.h 文件，重新编译 libxml2 即可应用此次裁剪。

如果在编译过程中有出现错误说 `int32_t int64_t uint64_t` 这类数据类型没有定义，请尝试复制 `win32/VC10` 目录下的 config.h 文件至源码根目录并覆盖同名文件，然后再尝试重新编译。

如果你发现编译生成的 libxml2 有 .exe 和 .lib 文件，请检查它的项目配置中的 `常规 > 配置类型` 这项的值是否为 `静态库(.lib)` 或 `动态库(.dll)`。

## 编译 LCUI

LCUI 主要是在 Windows 系统环境下开发的，你可以使用 [VisualStudio](https://www.visualstudio.com) 打开 `build/windows/LCUI.sln` 文件，在编译前，你需要将依赖项的库文件和头文件放在源码目录中的 vendor 目录中，目录结构如下：

    vendor/include
    vendor/lib

之后，在 VisualStudio 的菜单中选择 `生成 -> 生成解决方案` 来编译生成 LCUI。如果你用的是其它 IDE，请尝
试按该 IDE 的方式创建项目并将源文件添加至项目内，然后配置好依赖项再编译。

## 安装 LCUI

你可以建立一个专门的目录存放 LCUI 的库文件和头文件，例如：`D:/c-dev/`，但如果你的项目需要给其他人编译的话，我们建议你设置一个相对目录，例如在源码目录里建立一个 vendor 目录，这样其他人要编译你的项目时，只需要把 LCUI 库文件和头文件复制到 vendor 目录里，无需手动修改配置。

## 配置

每次新建 LCUI 应用项目时，你都需要手动为项目修改如下配置。看上去很麻烦，如果你感兴趣可以参考[这篇文章](https://msdn.microsoft.com/zh-cn/library/6db0hwky.aspx)为 VisualStudio 添加 LCUI 应用程序模板。

- **C/C++ > 常规 > 附加包含目录**

    将该项设置为：`$(SolutionDir)vendor\include`，如果你的 VisualStudio 工程文件不是建立在源码根目录下，例如：`/build/windows/project.sln`，那么请手动调整该配置项。

- **链接器 > 常规 > 附加库目录**

    将该项设置为：`$(OutDir)`，即：将 exe 输出目录作为附加库目录。

- **输入 > 附加依赖项**

    将该项设置为：`LCUI.lib; LCUIMain.lib`

- **生成事件 > 预先生成事件**

    将该项设置为：

        copy $(SolutionDir)vendor\lib\LCUI*.lib $(OutDir)
        copy $(SolutionDir)vendor\lib\LCUI*.dll $(OutDir)

    在编译、调试或运行应用时，需要让 lib、dll 文件和 exe 在一个目录，方便链接器能够找到它们。

### Windows 通用应用

如果你的应用是 Windows 通用应用，那么你需要在以上配置的基础上再做如下修改。

- **输入 > 附加依赖项**

    将该项设置为：`LCUI.lib; LCUIApp.lib`

- **生成事件 > 预先生成事件**

    将该项设置为：

        copy $(SolutionDir)vendor\lib\uwp\LCUI*.lib $(OutDir)
        copy $(SolutionDir)vendor\lib\uwp\LCUI*.dll $(OutDir)

    LCUI 的 Windows 通用应用版的库文件存放在 uwp 目录里。

- **文件**

    右键单击项目名称，然后选择 `添加 > 现有项...`，在文件选择器中选择 LCUI.dll 和 LCUIApp.dll 文件。添加后，右键点击这些文件并选择 `属性`，将 `常规 > 内容` 设置为 `是`。

## 完成

至此，你已经知道了 LCUI 的安装和编译方法，以及 LCUI 应用程序项目的配置方法，接下来你可以开始尝试[编写 Hello World](../getting_started/step1.html)。
