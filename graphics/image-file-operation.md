---
description: 介绍图像文件操作函数的用法。
---

# 图像文件操作

### 读取图像文件信息

使用图像读取器读取文件头中的信息：

```c
#include <stdio.h>
#include <LCUI/LCUI.h>
#include <LCUI/image.h>

int main(int argc, char *argv[])
{
    FILE *fp;
    LCUI_ImageReaderRec reader = { 0 };
    
    if (argc != 2) {
        printf("Please specify the image file path");
        return -1;
    }
    fp = fopen(argv[1], "rb");
    if (!fp) {
        printf("Cannot open file");
        return -2;
    }
    LCUI_SetImageReaderForFile(&reader, fp);
		if (LCUI_InitImageReader(&reader) != 0) {
		    printf("Unsupported image format");
        fclose(fp);
        return -3;
		}
		printf("image type: %d\n", reader.header.type);
		printf("image color type: %d\n", reader.header.color_type);
		printf("image size: %ux%u", reader.header.width, reader.header.height);
    LCUI_DestroyImageReader(&reader);
    fclose(fp);
    return 0;
}
```

这段代码会打开参数中指定的文件，然后初始化图像读取器，从文件中读取信息并在最后打印图像的类型、色彩类型和尺寸到屏幕上。

在使用 `fopen()` 打开文件后，调用 `LCUI_SetImageReaderForFile()` 将图像读取器的读取目标设置为标准文件流，然后调用 `LCUI_InitImageReader()` 初始化文件读取器，初始化时会尝试调用预置的图像读取接口检测该文件的格式。

### 读取图像数据

LCUI 提供了两个用于读取图像数据的接口：

```c
int LCUI_ReadImage(LCUI_ImageReader reader, LCUI_Graph *out);

int LCUI_ReadImageFile(const char *filepath, LCUI_Graph *out);
```

`LCUI_ReadImage()` 适用于从自定义的文件流中读取图像数据，在使用它之前你需要为图像读取器设置自定义文件流的句柄和操作函数。而 `LCUI_ReadImageFile()` 则适用于从标准文件流中读取图像数据，你只需要传入文件路径，它就能基于标准库的 `fopen()`、`fread()` 和 `fclose()` 等文件读取函数完成图像数据读取。

### 在读取时反馈进度

图像读取器的数据结构中的 [`fn_prog`](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/include/LCUI/image.h#L76) 成员变量是个函数指针，它会在每次读取完一行图像数据后调用，你只需要将该指针指向自定义函数即可获得进度信息。

基于上个示例代码，需要做这些改动：

```c
#include <stdio.h>
#include <LCUI/LCUI.h>
#include <LCUI/image.h>

void on_progress(void *data, float percent)
{
    printf("read %s: %.2f%%\n", data, percent);
}

int main(int argc, char *argv[])
{
    FILE *fp;
    LCUI_ImageReaderRec reader = { 0 };
    
    if (argc != 2) {
        printf("Please specify the image file path");
        return -1;
    }
    fp = fopen(argv[1], "rb");
    if (!fp) {
        printf("Cannot open file");
        return -2;
    }
    LCUI_SetImageReaderForFile(&reader, fp);
		if (LCUI_InitImageReader(&reader) != 0) {
		    printf("Unsupported image format");
        fclose(fp);
        return -3;
		}
		reader.fn_prog = on_progress;
		reader.prog_arg = argv[1];
		printf("image type: %d\n", reader.header.type);
		printf("image color type: %d\n", reader.header.color_type);
		printf("image size: %ux%u", reader.header.width, reader.header.height);
    LCUI_DestroyImageReader(&reader);
    fclose(fp);
    return 0;
}
```

### 自定义文件流

图像读取器的读取功能依赖 `fn_read` 、`fn_rewind` 和 `fn_skip` 这三个函数指针来实现，可供参考的最简单的例子是 [src/image/reader.c](https://github.com/lc-soft/LCUI/blob/345031d74ca65225ec3623e0c92d448f54f5052b/src/image/reader.c#L61-L83) 文件中的`LCUI_SetImageReaderForFile()` 函数，它对标准库的文件操作函数做了一层简单包装。如果你需要更加高级的用法，可以参考 LC Finder 项目的 [UWP/FileService.cpp](https://github.com/lc-soft/LC-Finder/blob/573f200698e2604450665716ebc6608837b4b73a/UWP/FileService.cpp#L611-L678) 文件中的图像文件读取器相关实现代码，它针对 UWP 的异步文件流做了适配。

### 将图像数据写入文件

由于图像写入功能很少用，因此并未拥有图像读取功能一般的完成度，目前能用的只有这一个函数：

```c
int LCUI_WritePNGFile(const char *file_name, const LCUI_Graph *graph);
```

