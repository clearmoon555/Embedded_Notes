[【快速入门 LVGL】-- 1、STM32 工程移植 LVGL-CSDN博客](https://blog.csdn.net/qq_49053936/article/details/136696700)

1、下载lvgl8.3，保存以下文件

![img](https://i-blog.csdnimg.cn/blog_migrate/fd02d903a84cdf7d006f5ba10bd2b3dd.png)

2、修改 lv_conf.h 文件名

在 "LVGL" 文件夹中，有 h文件："lv_conf_template.h"，是LVGL配置参数的重要文件。

原文件名：“lv_conf_template.h”，修改为： "lv_conf.h";
完成后， "LVGL" 文件夹，是这个样子的：

![img](https://i-blog.csdnimg.cn/blog_migrate/c9b295246e48b457ce3411fc77bd90c6.png)

**3、删除不需要的文件夹**

打开文件夹："**LVGL / examples":**

- 只保留 **porting** 文件夹，其它的文件夹和文件，都删除掉。

![img](https://i-blog.csdnimg.cn/blog_migrate/d660b792177557dec6960dd0f88fb137.png)

**4、修改 porting 里面的文件名称**

打开刚才的 "porting" 文件夹：

- 6个文件的名称，都删除 "_template" 字样

完成后，"porting" 文件夹，是这个样子的：![img](https://i-blog.csdnimg.cn/blog_migrate/cb6fb8ba507721cd57a742f6ecc2be28.png)

**5、启用 lv_conf.h**

双击打开 lv_conf.h，对以下内容进行修改，以启用此文件。

- 第15行，原：#if 0，修改为：#if 1 

完成后，是这个样子的：![img](https://i-blog.csdnimg.cn/blog_migrate/0931cf81a1efc0cd3aa484d14016a1d1.png)

**6、启用 lv_port_disp.h**

双击打开 lv_port_disp.h，修改以下内容，以启用此文件：

- 第7行，原：#if 0, 修改为：#if 1 
- 第22行，原：“lvgl/lvgl.h", 修改为：”lvgl.h"

完成后，是这个样子的：![img](https://i-blog.csdnimg.cn/blog_migrate/40cfe92aef63bc165e91f253bbb7c425.png)

7、启用  lv_port_disp.c 

双击打开 lv_port_disp.c，修改以下内容，以启用此文件：

第7行，原：#if 0, 修改为：#if 1 
第12行，原"lv_port_disp_template.h", 修改为："lv_port_disp.h"
完成后，是这个样子的：
![img](https://i-blog.csdnimg.cn/blog_migrate/8adfe8ebe9c390c97e7a894c503746db.png)

8、添加 LCD 驱动的头文件

在 lv_port_disp.c中：

第14行，插入你的LCD驱动文件，如：#include "bsp_LCD_ILI341.h"，写上你的h文件。
第20行、第25行，是显示屏的宽、高度。查看你的显示屏参数，填写实际像素即可。
插入LCD的头文件，目的是为了让这个c文件，能调用LCD的: 画点函数;

注意一个：LVGL默认使用横屏的方式，这一点要注意，别写反了。

完成后，是这个样子的
![img](https://i-blog.csdnimg.cn/blog_migrate/c8318f01c76247e640f2150877f88703.png)

9、选择创建缓存的方式，3选1

还是在 lv_port_disp.c 中，向下滚动，

（会出现很多错误提示，不用管。也可以先编译一次，让刚才启用的h文件生效，错误就会消失）

第86行到101行，LVGL 提供了创建显示缓冲区的3种方式，这里，必须3选1。

绝大多数情况下，使用第1种方法，即：只创建1个缓冲区;

注释掉第90~101行，即：不使用第2和第3种方法;
完成后，是这个样子的：
![img](https://i-blog.csdnimg.cn/blog_migrate/51fed2818c8f3c0b0e8df1333549eb13.png)

10、关联 画点函数

还是在 lv_port_disp.c 中，向下滚动，找到disp_flush( )函数：

第173行，替换你的 LCD 的画点函数;  参数：x坐标、y坐标、16位颜色值。
小编用的画点函数：LCD_DrawPoint( x, y, color_p->full) ;
你的画点函数，可能和小篇所用的不一样，照样画瓢即可。

完成后，是这个样子的：
![img](https://i-blog.csdnimg.cn/blog_migrate/6f992ec48e0920a7b392b9b01b4b1b5b.png)

或者

![img](https://i-blog.csdnimg.cn/blog_migrate/deae4351396e284aa1ee378f28b8e2da.png)

11、显示

```c
//初始化
LCD_Init();
lv_init();
lv_port_disp_init();
  
//清屏为白色背景
LCD_Fill(0, 0, LCD_W, LCD_H, WHITE); 

//设置背光
LCD_Open_Light();        
LCD_Set_Light(20);      
  
//加载页面到默认屏幕
lv_scr_load(ui_HRPage);
```



