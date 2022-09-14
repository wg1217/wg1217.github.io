原文来自以下网站，本文只做记录，方便日后寻找

 https://www.jianshu.com/p/949508a0b22d

 https://jingyan.baidu.com/article/2f9b480df81cb941cb6cc217.html



# 问题描述

win+r打开`运行`，搜索`gpedit.msc`，win10提示找不到该应用程序。





# 解决办法

# 方法一

 1、新建文本文档，将下面代码复制到文本文档中：

```dart
@echo off
pushd "%~dp0"
dir /b C:\Windows\servicing\Packages\Microsoft-Windows-GroupPolicy-ClientExtensions-Package~3*.mum >List.txt
dir /b C:\Windows\servicing\Packages\Microsoft-Windows-GroupPolicy-ClientTools-Package~3*.mum >>List.txt
for /f %%i in ('findstr /i . List.txt 2^>nul') do dism /online /norestart /add-package:"C:\Windows\servicing\Packages\%%i"

pause
```

复制代码的时候要注意，一行一行的复制。直接粘贴复制的话代码与代码之间没有换行。



2、将文件重命名为：组策略.bat，一定要是.bat批处理格式。



3、运行该批处理文件即可。如果运行时提示权限错误，需右键文件选择“以管理员身份运行”





# 方法二 (未测试，方法一解决问题)

1. 如图所示 打开组策略报错这个东东

   ![win10找不到组策略gpedit.msc怎么办](https://raw.githubusercontent.com/wg1217/picbed/main/202209132354525.jpg)

   

2. 按住快捷键Win+R打开运行窗口，输入“regedit”，这样就打开了注册表编辑器，如图：

   ![win10找不到组策略gpedit.msc怎么办](https://raw.githubusercontent.com/wg1217/picbed/main/202209132354526.jpg)

   

3. 点开注册表

   ![win10找不到组策略gpedit.msc怎么办](https://raw.githubusercontent.com/wg1217/picbed/main/202209132354527.jpg)

   

4. 在编辑器左侧依次找到HKEY_CURRENT_USER\Software\Policies\Microsoft\MMC，然后将RestrictToPermittedSnapins的值设置为0，如图。

   ![win10找不到组策略gpedit.msc怎么办](https://raw.githubusercontent.com/wg1217/picbed/main/202209132354528.jpg)

   

5. 把这里的值改成 0

   ![win10找不到组策略gpedit.msc怎么办](https://raw.githubusercontent.com/wg1217/picbed/main/202209132354529.jpg)

   

6. 但是有的朋友按照以上方法却找不到MMC，这时我们就要新建一个文本文档了，然后将以下内容复制到记事本。

   ```
   Windows Registry Editor Version 5.00[HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Group Policy Objects\LocalUser\Software\Policies\Microsoft\MMC][-HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Group Policy Objects\LocalUser\Software\Policies\Microsoft\MMC\{8FC0B734-A0E1-11D1-A7D3-0000F87571E3}]"Restrict_Run"=dword:00000000[HKEY_CURRENT_USER\Software\Policies\Microsoft\MMC][-HKEY_CURRENT_USER\Software\Policies\Microsoft\MMC\{8FC0B734-A0E1-11D1-A7D3-0000F87571E3}]"Restrict_Run"=dword:00000000[HKEY_CURRENT_USER\Software\Policies\Microsoft\MMC]"RestrictToPermittedSnapins"=dword:00000000
   ```

   ![win10找不到组策略gpedit.msc怎么办](https://raw.githubusercontent.com/wg1217/picbed/main/202209132354530.jpg)

   

7. 复制粘贴以后，将文件后缀名更改为.reg，点击运行以后就可以找到了，重启电脑以后再次输入gpedit.msc命令就可以打开组策略了，大家可以试试。

   ![win10找不到组策略gpedit.msc怎么办](https://raw.githubusercontent.com/wg1217/picbed/main/202209132354531.jpg)。

   