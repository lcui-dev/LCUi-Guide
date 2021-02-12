# 图像文件操作

### 读取图片文件信息

```c
#include <stdio.h>
#include <LCUI/LCUI.h>
#include <LCUI/image.h>

int main(int argc, char *argv[])
{
    FILE *fp;
    LCUI_ImageHeaderRec reader = { 0 };
    
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

### 读取图片文件数据



