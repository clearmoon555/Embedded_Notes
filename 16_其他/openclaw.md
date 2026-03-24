一、安装Node.js

二、安装openclaw

1.以管理员身份打开 PowerShell

2.允许脚本执行

```
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser 
# 提示时输入 Y 回车
```

3.执行一键安装脚本

```
# 官方脚本（推荐） 
iwr -useb https://openclaw.ai/install.ps1 | iex 
# 国内网络慢可换社区镜像
iwr -useb https://open-claw.org.cn/install-cn.ps1 | iex
```

4.初始化和启动

```
# 初始化配置
openclaw onboard --flow quickstart
# 启动
openclaw gateway start
# 重启网关 
openclaw gateway restart 
# 查看状态 
openclaw status 
# 停止服务 
openclaw gateway stop
# 获取网页地址
openclaw dashboard --no-open
```

三、打开chrome浏览器

```
"C:\Program Files\Google\Chrome\Application\chrome.exe" ^
--remote-debugging-port=9222 ^
--user-data-dir="C:\temp\chrome-debug"
```

```
# 检查
netstat -ano | findstr 9222 
```

