界面说明
界面说明主要看图：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9db4d78b9c4f3b59854be9b4d99982d3.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/fdae5555ce56f9c1119d5f4c8317d9b1.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/8ed56ca90767e46606a909fcc77babd0.png)![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/77f74c81a4339071c5e8e2e42c4bb509.png)![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/75a3b9bb05f1ee4a6de212f46d93fdc4.png)![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/21da17fbe6d9e9f90bc49986c5111355.png)项目文件夹
项目文件夹中重要文件说明如下：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/61bccc6fc228cc9423bafeddcea9121d.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/c47e4338994f0c4416799ec358e8fd87.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/b381b16821127a4eaab182bc18c8ef8d.png)

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/aebd57fc768d0d482aca2fe0645810e1.png)常用快捷键
STM32CubeIDE快捷键很多，可以通过 Help > Show Active Keybindings... 查看当前可用快捷键；也可以在 Window > Preferences > General > Keys 中查看修改快捷键，默认的常用快捷键如下（一些特别常用的就不说了，比如Ctrl+Z、Ctrl+Y、Ctrl+C、Ctrl+V这些)：

快捷键	快捷键说明
Ctrl+/	注释行/取消注释行
Ctrl+D	删除行
Ctrl+Shift+F	格式化代码
Alt+/	代码补全（这个对于现在程序员来说算是重要功能了吧）
Shift+Enter	在当前行的下一行插入空行
Alt+↓/↑	行下移/上移（可按住）
Ctrl+↑/↓	编辑器视图上移/下移（可按住）
Alt+←/→	前一个/后一个页面
F3	跳转到声明处
Ctrl+F	文件内内搜索
Ctrl+H	项目内搜索
Ctrl+M	最大化/默认当前窗口
Ctrl+L	跳转至某行
Ctrl+O	显示大纲（方便跳转）
Ctrl+W	关闭当前窗口
F11	启动调试
F5	单步跳入（调试时）
F6	单步跳过（调试时）
F7	单步返回（调试时）
F8	继续运行（调试时）
开发流程
开发流程这里主要指和这个软件相关的，主要会涉及到的流程如下（根据项目实际情况可能会有不同）：

新建项目（有多处可以新建项目，通用方式 File > New > STM32 Project ）：


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/1d911d7bdfb8ec839bf4083bf2859151.gif)配置芯片资源并生成初始化代码（操作的是 项目名.ioc 文件)：


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/5edf80b1d7c8c09971b2e12b6c408b26.gif)代码编写与调试（是否调试、怎么调试看个人习惯和需求)：


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/86084b090f59a63446e2a4eb858c1d95.gif)生成给芯片烧录用的文件（Release版本： 项目文件夹 > Release文件夹 > 项目名.elf )：


![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/cc1692f6344a9051d358fbd55644630a.gif)烧录程序到目标芯片（使用计算机软件烧录的话可以用ST官方的工具 STM32CubeProgrammer ，可以从官网搜索下载)：

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/889a2f781864e9a7b810d033d8326aa2.gif)