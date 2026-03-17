## 一、安装（以下最好使用管理员权限打开命令行）

1.安装zephyr依赖工具，打开命令行运行以下命令

```
winget install Kitware.CMake Ninja-build.Ninja oss-winget.gperf Python.Python.3.12 Git.Git oss-winget.dtc wget 7zip.7zip
```

2.新建文件夹，在新目录下创建python虚拟环境，命令如下

```
python -m venv zephyrproject\.venv
```

3.激活虚拟环境

```
zephyrproject\.venv\Scripts\activate
```

4.安装west

```
pip install west
```

5.下载zephyr源码

```
west init zephyrproject
cd zephyrproject
west update
west zephyr-export
pip install -r zephyr/scripts/requirements.txt
west sdk install
```

6.设置 SDK 路径环境变量 

```
$env:ZEPHYR_SDK_INSTALL_DIR = "C:\Users\0\zephyr-sdk-0.17.4"
```

7.编译

```
west build -b esp32_devkitc/esp32/procpu zephyr/samples/basic/blinky_pwm --pristine
```

8.烧录

```
west flash
```

9.监听

```
west espressif monitor
```

## 二、应用开发

#### 创建应用

1.进入你的工作区：

```
cd zephyrproject
```

2.克隆模板：

```
git clone https://github.com/zephyrproject-rtos/example-application my-app
```

#### 配置文件

prj.conf 配置的是 **软件**（宏定义、功能开关）。

app.overlay 配置的是 **硬件**（引脚映射、外设节点）。

在项目根目录下创建zephyr\samples\basic\blinky\app.overlay，编译时会直接包含这个文件

.overlay是.conf的补充文件，.overlay和.conf最后被转化为.h文件中的宏定义，从而控制.c文件中条件编译是否通过，另外控制某些文件是否被编译。

