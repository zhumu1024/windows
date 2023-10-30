# windows
The knowledge of windows system 

## 大块代码学习及注释记录---生信学习流程--[Setting-up scripts for Windows 11](https://github.com/wang-q/windows/blob/master/README.md#symlinks)
此部分处暂时记录大块的代码及注释
### sysinternals板块学习


Sysinternals 是一组由微软公司开发的系统工具和实用程序集合，专为 Microsoft Windows 操作系统设计。这些工具提供了许多`高级系统管理和诊断功能`，可用于`分析、监控和维护 Windows 系统`。

*powershell 

```powershell
[Environment]::SetEnvironmentVariable(
    # [Environment]::SetEnvironmentVariable("Path", ...)：这是一个 PowerShell 命令，用于设置环境变量的值。包括三个参数，要设置的环境变量的名称，值和目标。
    "Path",
    #环境变量的值
    [Environment]::GetEnvironmentVariable("Path", [EnvironmentVariableTarget]::Machine) + ";$HOME\bin",
    #环境变量的新值
    #[EnvironmentVariableTarget]::Machine)：这部分用于获取系统环境变量 Path 的当前值。
    #  ";$HOME\bin",添加到 Path 环境变量中的路径。它包括用户主目录下的 bin 目录
    # [Environment] 一个类，包含各种与环境变量和系统信息
    # :: 静态访问运算符，用于访问类的静态成员
    #[EnvironmentVariableTarget]::Machine返回了系统环境变量 Path 的当前值 这是一个字符串
    [EnvironmentVariableTarget]::Machine)
    #环境变量的目标

```

```powershell
scoop install aria2 unzip

$array = "DU", "ProcessExplorer", "ProcessMonitor", "RAMMap"
#定义一个字符串数组
foreach ($app in $array)
{
    iwr "https://download.sysinternals.com/files/$app.zip" -O "$app.zip"
    # iwr Invoke-WebRequest  在 PowerShell 中执行网络请求，通常用于下载文件或通信
    # -O 指定要将下载的内容保存到本地文件时的文件名或路径
}

foreach ($app in $array)
{
    unzip "$app.zip" -d $HOME/bin -x Eula.txt
    # unzip 解压缩
    # -d 解压缩到的目录
    # -x 后面所跟文件不解压缩
    #在大多数情况下可以不加双引号，但最佳实践是将文件名用双引号括起来，以确保脚本在各种情况下都能正确处理文件名
}

rm $HOME/bin/*.chm
#
rm $HOME/bin/*64.exe
rm $HOME/bin/*64a.exe

```
### QuickLook Plugins板块代码学习

```powershell
# epub
$url = (
#声明变量可以提高代码的可读性、可维护性和错误检查，使代码更清晰和可靠。这是一种良好的编程实践。
iwr https://api.github.com/repos/QL-Win/QuickLook.Plugin.EpubViewer/releases/latest |
# 管道将下载的数据交给后面的命令
    Select-Object -Expand Content |
#Select-Object -Expand Content：这部分从 API 响应中选择了 Content 字段的内容。Content 字段通常包含有关发布的详细信息，包括发布的附件（assets）信息。
#-Expand 将对象的内容展开
    jq -r '.assets[0].browser_download_url'
# jq 工具解析 JSON 数据
#-r 以原始（raw）输出格式返回结果
#. 表示当前上下文的对象
#.browser_download_url：从选定的数组元素中选择了名为 browser_download_url 的字段
)
curl.exe -LO $url
#-O全称是 --location。它表示要遵循 HTTP 重定向，如果下载的文件是一个 HTTP 链接，并且该链接被重定向到另一个位置，下载最终的文件
#-L全称是 --remote-name。它表示将下载的文件保存为与远程文件相同的名称
```
