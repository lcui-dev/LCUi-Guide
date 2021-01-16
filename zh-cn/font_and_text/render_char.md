# 渲染一个字

以下程序的功能是渲染一个红色的“字”并将它保存至 PNG 文件中，使用的字体为宋体。

```c
#include <LCUI_Build.h>
#include <LCUI/LCUI.h>
#include <LCUI/graph.h>
#include <LCUI/font.h>

int main( void )
{
        int ret, fid;
        LCUI_Graph img;
        LCUI_FontBitmap bmp;
        LCUI_Pos pos = {25, 25};

        /* 初始化字体处理功能 */
        LCUI_InitFont();

        /* 创建一个图像，并使用灰色填充 */
        Graph_Init( &img );
        Graph_Create( &img, 100, 100 );
        Graph_FillRect( &img, RGB( 240, 240, 240 ), NULL, FALSE );

        /* 载入字体文件 */
        ret = LCUIFont_LoadFile( "C:/Windows/fonts/simsun.ttc" );
        while( ret == 0 ) {
                /* 获取字体ID */
                fid = LCUIFont_GetId( "SimSun", NULL );
                if( fid < 0 ) {
                        break;
                }
                /* 载入对应的文字位图，大小为 48 像素 */
                ret = FontBitmap_Load( &bmp, L'字', fid, 48 );
                if( ret != 0 ) {
                        break;
                }
                /* 绘制红色文字到图像上 */
                FontBitmap_Mix( &img, pos, &bmp, RGB( 255, 0, 0 ) );
                Graph_WritePNG( "test_char_render.png", &img );
                /* 释放内存资源 */
                FontBitmap_Free( &bmp );
                Graph_Free( &img );
                break;
        }

        /* 退出字体处理功能 */
        LCUI_ExitFont();
        return ret;
}
```

编译运行后，可以在程序所在工作目录下找到 test\_char\_render.png 文件，打开它可看到如下图所示的内容：

![&#x7ED8;&#x5236;&#x51FA;&#x6765;&#x7684;&#x6587;&#x5B57;](../../.gitbook/assets/test_char_render.png)

在绘制文字时都需要指定字体的 ID，这个 ID 标识了字体的字族和风格，如果要使用默认的字体，可以使用 -1 作为 ID。字体位图数据是 `LCUI_FontBitmap` 类型，可以用 `FontBitmap_Mix()` 函数将该字体位图绘制到指定的图像上，该函数支持自定义字体颜色。

