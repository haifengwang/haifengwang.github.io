---
layout: post
title: PowerShell 让开发更高效
category: others
description: 《卓有成效的程序员》中提到如何在开发软件的过程中变得更加高效。高效的方式有好多种，比如说用好的工具，好的管理方式，好的习惯等。善用shell脚本就是其中之一。
keywords: 开发,powersehll,效率,常用
--- 

中午一位同学删除一个老项目中的`.svn`文件。由于各层目录比较多，就用 `C#` 写了一段程序，程序写好后发现，由于权限问题删除不了。没有办法该同学请教另一位同学。另一位同学在之前的程序的基础上调用 **CMD** 批处理的方式搞定（这是一个不错的方式，直接写批处理也可以，只是遍历和筛选文件，不熟悉批处理的人很不容易）。

我尝试着用`PowerShell` 写了一段脚本。效果很不错，因为 **PowerShell** 支持强制删除。
代码：

```shell
//删除文件
$path = "D:\Test"
Get-ChildItem $path -Recurse -Force -Include .svn | remove-item -Force -Recurse 
```

像 PowerShell 这种脚本，如果熟练了，在工作中会很方便的。曾听过这样一则故事，老板让一个程序员监控系统运行的情况。一个C语言程序员想着先做一个类似的监控系统，另一个用了一句 `shell` 代码就搞定了。

《卓有成效的程序员》提到如何在开发软件的过程中变得更加高效。高效的方式有好多种，比如说用好的工具，好的习惯等。用 **shell** 也一定会开发变的更加高效。


一些常用的**Power Shell** 命令。

+ 入门级别

1. 在文件里递回地搜索某个字符串 `dir –r | select string "searchforthis" `

2. 使用内存找到五个进程 `ps | sort –p ws | select –last 5`

4. 循环（停止，然后重启）一个服务，如DHCP `Restart-Service DHCP`

5. 在文件夹里列出所有条目 `Get-ChildItem – Force`

6. 递归一系列的目录或文件夹 `Get-ChildItem –Force c:/directory –Recurse`

7. 在目录里移除所有文件而不需要单个移除 `Remove-Item C:/tobedeleted –Recurse`

8. 重启当前计算机 

```
Get-WmiObject -Class Win32_OperatingSystem -ComputerName .).Win32Shutdown
```

+ 收集信息

9. 获取计算机组成或模型信息 `Get-WmiObject -Class Win32_ComputerSystem`

10. 获取当前计算机的BIOS信息 `Get-WmiObject -Class Win32_BIOS -ComputerName`

11. 列出所安装的修复程序（如QFE或Windows Update文件）

 ```Get-WmiObject -Class Win32_QuickFixEngineering -ComputerName```

12. 获取当前登录计算机的用户的用户名 

```
Get-WmiObject -Class Win32_ComputerSystem -Property UserName -ComputerName
```

13. 获取当前计算机所安装的应用的名字 

```
Get-WmiObject -Class Win32_Prod t -ComputerName . | Format-Wide -Column 1
```

14. 获取分配给当前计算机的IP地址 

```
Get-WmiObject -Class Win32_NetworkAdapterConfiguration -Filter IPEnabled=true -ComputerName . | Format-Table -Property IPAddress
```

15. 获取当前机器详细的IP配置报道 

```
Get-WmiObject -Class Win32_NetworkAdapterConfiguration -Filter IPEnabled=true -ComputerName . | Select-Object -Property [a-z]* -Excl?Property IPX*,WINS*
```

16. 找到当前计算机上使用DHCP启用的网络卡 

```
Get-WmiObject -Class Win32_NetworkAdapterConfiguration -Filter "DHCPEnabled=true " -ComputerName
```

17. 在当前计算机上的所有网络适配器上启用 

```
DHCP——Get-WmiObject -Class Win32_NetworkAdapterConfiguration -Filter IPEnabled=true -ComputerName . | ForEach-Object -Process {$_.EnableDHCP()}
```

+ 软件管理

18. 在远程计算机上安装MSI包 

```
Get-WMIObject -ComputerName TARGETMACHINE -List | Where-Object -FilterScript {$_.Name -eq "Win32_Prod t"}).Install(//MACHINEWHEREMSIRESIDES/path/package.msi

```

19. 使用基于MSI的应用升级包升级所安装的应用 

```
Get-WmiObject -Class Win32_Prod t -ComputerName . -Filter "Name='name_of_app_to_be_upgraded'").Upgrade(//MACHINEWHEREMSIRESIDES/path/upgrade_package.msi

```

20. 从当前计算机移除MSI包 

```
Get-WmiObject -Class Win32_Prod t -Filter "Name='prod t_to_remove'" -ComputerName . ).Uninstall()

```

+ 机器管理

21. 一分钟后远程关闭另一台机器 

```
Start-Sleep 60; Restart-Computer –Force –ComputerName TARGETMACHINE
```

22. 添加打印机 

```
New-Object -ComObject WScript.Network).AddWindowsPrinterConnection(//printerserver/hplaser3
```

23. 移除打印机 

```
New-Object -ComObject WScript.Network).RemovePrinterConnection("//printerserver/hplaser3 ")
```

24. 进入PowerShell会话 

```
invoke-command -computername machine1, machine2 -filepath c:/Script/script.ps1
```