# windows
The knowledge of windows system 

## 大块代码学习及注释记录---生信学习流程--[Setting-up scripts for Windows 11](https://github.com/wang-q/windows/blob/master/README.md#symlinks)
此部分处暂时记录不便记载在笔记本上的大块的代码的学习及注释
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
### ps1文件代码学习
PS1 文件是 Windows PowerShell 脚本文件的扩展名，包含了一系列的 PowerShell 命令和脚本，可以执行各种任务，从文件操作到系统管理和配置。

```
Function RequireAdmin {
# 定义一个函数
	If (!([Security.Principal.WindowsPrincipal][Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole]"Administrator")) {
    # if条件语句
    # ！（）对返回的结果取反
    #[Security.Principal.WindowsPrincipal]是返回了用户的身份信息和角色，[Security.Principal.WindowsIdentity]::GetCurrent())则是取出其中当前用户的身份信息，Security 和 Principal 都是命名空间（Namespace）的一部分，用 
    于组织 .NET Framework 中的类。
    #.表示访问类的静态成员。它不用于参数的传递，而是用于从指定的类中访问静态方法或属性等成员。
    #.IsInRole([Security.Principal.WindowsBuiltInRole]"Administrator" 判断当前角色是否属于Administrator
    
		Start-Process powershell.exe "-NoProfile -ExecutionPolicy Bypass -File `"$PSCommandPath`" $PSCommandArgs" -Verb RunAs
        #-Verb RunAs 是 Start-Process cmdlet 的一个参数，用于指定启动进程时要使用的操作动词。在此处，它用于指定以管理员权限运行新的进程。
            #在 Windows 操作系统中，动词通常用于指定要对文件或进程执行的操作。例如，"打开"、"编辑"、"运行" 等都是动词，它们表示执行不同的操作。unAs是一个特定的动词，表示以不同的用户身份或权限运行程序。
		Exit
        #命令执行后，退出
	}
}
```

```
$tweaks = @()
$PSCommandArgs = @()
#定义两个空白数组，如果之后需要将值添加到 $tweaks 或 $PSCommandArgs 中，可以通过使用 += 运算符将值添加到数组中
```


```
Function AddOrRemoveTweak($tweak) {
#创建一个AddOrRemoveTweak的函数，可以接受$tweak作为参数
	If ($tweak[0] -eq "!") {
    #-eq等于
		$script:tweaks = $script:tweaks | Where-Object { $_ -ne $tweak.Substring(1) }
        #这句表示，$tweak的值重置为 tweak | Where-Object { $_ -ne $tweak.Substring(1) }的结果
        # 的优先级较低，$_ 表示正在处理的数组元素 -ne： 不等于
        # $tweak.Substring(1)  取$tweak的子字符串  （1）表示从第二位开始取
	} ElseIf ($tweak -ne "") {

		$script:tweaks += $tweak
        #在$script:tweaks数组中添加新的元素
	}
}

```
```
$i = 0
#设置一个变量，并赋值为0
While ($i -lt $args.Length) {

	If ($args[$i].ToLower() -eq "-include") {
    #.ToLower()  将字符串全部变成小写
		
		$include = Resolve-Path $args[++$i] -ErrorAction Stop
        #Resolve-Path  将相对路径或者绝对路径转换成绝对路径
        #$args[++$i] 表示将索引 $i 递增并使用递增后的值来获取下一个参数，即文件路径。
        #-ErrorAction Stop 表示如果无法解析路径，将停止脚本的执行。
		$PSCommandArgs += "-include `"$include`""
        #将 -include 参数和解析后的文件路径包含在 $PSCommandArgs 数组中。这是为了将参数传递给脚本的后续部分。注意，参数值用双引号括起来，以处理文件路径中可能包含空格等特殊字符的情况
		
		Import-Module -Name $include -ErrorAction Stop
        #Import-Module 是 PowerShell 中的命令，用于导入一个或多个模块。模块是一种包含了一组相关功能和命令的封装单元，它们可以被重复使用和共享。Import-Module -Name ModuleName

	} ElseIf ($args[$i].ToLower() -eq "-preset") {
		
		$preset = Resolve-Path $args[++$i] -ErrorAction Stop
		$PSCommandArgs += "-preset `"$preset`""
		
		Get-Content $preset -ErrorAction Stop | ForEach-Object { AddOrRemoveTweak($_.Split("#")[0].Trim()) }

        #Get-Content 是 PowerShell 命令，用于获取文件的内容。$preset 是变量，包含了要读取内容的文件路径。

	#ForEach-Object：这是 PowerShell 中的命令，用于遍历一个集合（在这种情况下，是从文件中读取的行），并对每个元素执行指定的操作。
	#{ }用于定义一个脚本块
	#ForEach-Object：这是 PowerShell 中的命令，用于遍历一个集合，并对每个元素执行指定的操作。
	#.Split("#")：这是一个字符串方法，将当前行文本按 # 字符拆分成多个部分。# 是注释字符，所以这一步实际上将删除注释部分。[0]：这表示从拆分后的数组中取第一个部分，即去掉注释部分后的文本。.Trim()：这是字符串方法，用于删除文本两侧的空白字符（如空格、制表符等）。

```


