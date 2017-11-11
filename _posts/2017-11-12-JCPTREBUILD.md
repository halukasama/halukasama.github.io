---
layout:     post
title:      "NSIS脚本教程整理"
subtitle:   "NSIS script"
date:       2017-09-25 12:00:00
author:     "Wjl"
header-img: "img/post_jcptrebuild.jpg"
catalog: true
tags:
    - NSIS
    - 脚本
    - 转载
---

## 在新工作中深刻领会了[破窗效应](https://zh.wikipedia.org/wiki/%E7%A0%B4%E7%AA%97%E6%95%88%E5%BA%94)的可怕。

如果追求进度而可以忽略代码质量。那么我可能要成为一辈子的猴子了。  
会在下周开始有计划的对当前项目进行整改。
另外要有计划的学习NSIS打包脚本了。
这里就简单作一下汇总吧


[NSIS.wiki](http://nsis.sourceforge.net/Wiki_Information) --NSIS sourceforge的wiki页。

[NSIS.gitfork](https://github.com/kichik/nsis)
--自己fork的源码 有时间可以读读。

[NSIS.faq](http://nsis.sourceforge.net/FAQ) 

常用示例
[原文地址](http://www.cnblogs.com/myall/p/3637759.html)

nsis中文版（Nullsoft Scriptable Install System）是一个专业的开源的可以用来封闭Windows程序的实用工具,  
是一个开源的 Windows 系统下安装程序制作程序。  
nsis它功能强,源码是直接使用C语言编写而成(`我自己看源码显示有33%的c++`),并且可以直接到nsis官网下载所有nsis版本,并且提供了详细的帮助文档,方便用户制作时使用。    
安装页面可以使用Page自定义界面、卸载页面可以配置让用户选择是否删除用户文件、系统设置中对注册表的操作简单方便,  
可以使用REGDLL进行动态库的注册,还可以使用诸如EXECWAIT来执行外部程序,  
还可以用SC等命令或插件来完成服务注册安装、提供lzma固实文件解压缩极强的压缩性能尤为突出,并且提供上百的插件扩展使得程序编写更加得心应手,  
在封装功能,速度,稳定性等方便更胜一筹。


### 1 代码注释

在nsis中可以使用单行注释，和多行注释,多行注释不支持嵌套。:#,;,/**/

```
#OutFile "注释.exe"
;Section ""
/*
    MessageBox MB_OK "Hello World!"
SentionEnd
*/
```

### 2 转义字符和续行符

转意字符用"$\"作前缀。美元符号、常用转意字符换行、回车、制表符的nsi语法表示分别为：$$,$\n,$\r,$\t

nsi脚本用行尾的反斜杠"\"表示下一行和当前行逻辑上是同一行

### 3 互斥运行

调用系统API创建互斥进程
```
OutFile "Temp.exe"
Section "Temp"
SectionEnd
Function .onInit
InitPluginsDir
  ;创建互斥防止重复运行
  System::Call 'kernel32::CreateMutexA(i 0, i 0, t "Temp") i .r1 ?e'
  Pop $R0
  StrCmp $R0 0 +3
    MessageBox MB_OK|MB_ICONEXCLAMATION "The program is runing!"
    Abort
FunctionEnd
```

### 4 7-Zip 打开空白

实现7-Zip打开空白


!system '>blank set/p=MSCF<nul'
!packhdr temp.dat 'cmd /c Copy /b temp.dat /b +blank&&del blank'


如果你想增加一个有内容的7z压缩到可执行文件头部，那么在脚本开始位置增加下面这行代码就可以了：

!packhdr temp.dat 'cmd /c Copy /B temp.dat /B +外链压缩包.7z temp.dat'
### 5 压缩资源头

压缩资源，直接用资源查看器会提示已加壳，需要先脱壳才能查看资源

!packhdr "exehead.tmp" '"upx.exe" -4 "exehead.tmp"'
### 6 关于nsis中的变量使用
```
#程序输出名称
OutFile "Variables.exe"
#设置压缩方式
SetCompress force
SetCompressor /SOLID lzma
#是否进入调试模式
#!define _DEBUG ""
#安装目录
!define InstDir "$PROGRAMFILES\Variables"
#压缩资源
!packhdr "exehead.tmp" '"upx.exe" -4 "exehead.tmp"'
#区段
Section "nsis变量"
    #在安装过程中改变安装目录
    StrCpy "$INSTDIR" "$EXEDIR\Variables"
    #设置输出目录
    SetOutPath "$INSTDIR"
    #显示输出目录
    DetailPrint "$OUTDIR"
    #将.(当前目录)打包和程序,安装目录为$INSTDIR
    File /r ".\*.nsi"
    File /r ".\*.7z" 
    
    #输出当前主程序目录
    DetailPrint "$EXEDIR"
    #输出当前使用的语言标记符
    DetailPrint "输出当前使用的语言标记符$LANGUAGE"
 
#"nsis命令行参数操作"
Call nsis_cmdline
SectionEnd
 
Function nsis_cmdline #"nsis命令行参数操作"
#使用变量$CMDLINE获得命令行信息
    !ifdef _DEBUG
    MessageBox MB_OK|MB_ICONINFORMATION "$CMDLINE"
    !endif
    DetailPrint "$CMDLINE"
#使用宏来处理命令行信息,它也是基于对$CMDLINE的信息进行处理
    !include FileFunc.nsh
    ${GetParameters} $R0 # 获得命令行
    ${GetOptions} $R0 "/T"  $R0 # 在命令行里查找是否存在/S选项
    !ifdef _DEBUG
    IfErrors 0 +2
    MessageBox MB_OK "Not found" IDOK +2
    MessageBox MB_OK "Found"
    !endif
    IfErrors 0 +3
    DetailPrint "Not found"
    goto +2
    DetailPrint "Found"
FunctionEnd
这些变量也可以在插件里传递，因为他们可以被 DLL 插件读取和写入
enum
{
INST_0,         // $0
INST_1,         // $1
INST_2,         // $2
INST_3,         // $3
INST_4,         // $4
INST_5,         // $5
INST_6,         // $6
INST_7,         // $7
INST_8,         // $8
INST_9,         // $9
INST_R0,        // $R0
INST_R1,        // $R1
INST_R2,        // $R2
INST_R3,        // $R3
INST_R4,        // $R4
INST_R5,        // $R5
INST_R6,        // $R6
INST_R7,        // $R7
INST_R8,        // $R8
INST_R9,        // $R9
INST_CMDLINE,   // $CMDLINE
INST_INSTDIR,   // $INSTDIR
INST_OUTDIR,    // $OUTDIR
INST_EXEDIR,    // $EXEDIR
INST_LANG,      // $LANGUAGE
__INST_LAST
};
*/
```
### 7 关于nsis中的界面制作
```
#头文件
!include nsDialogs.nsh
#设置压缩方式
SetCompressor /SOLID lzma
#支持XP风格
XPStyle on
#标题栏,输出文件,图标
Caption "NSIS自定义界面中文版教程"
OutFile "NSIS自定义界面中文版教程.exe"
#变量定义
Var Dialog
Var Image1
#页面
Page custom page1
#初始化函数
Function .onInit
InitPluginsDir
SetOutPath "$PLUGINSDIR"
File "Image\*.*"
FunctionEnd
#页面函数
Function page1
nsDialogs::Create /NOUNLOAD 1018
Pop $Dialog
${If} $Dialog == error
Abort
${EndIf}
${NSd_CreateBitmap} 0 0 100% 100% '全屏图片'
Pop $Image1
${NSD_SetImage} $Image1 '$PLUGINSDIR\bg.bmp' $1
nsDialogs::show
FunctionEnd
Section "NSIS自定义界面中文版教程"
SectionEnd
```
### 8 关于nsis中的字符串处理
```
!include "WordFunc.nsh"
OutFile "关于nsis中的字符串处理.exe"
Var stemp
Section "关于nsis中的字符串处理"
StrCpy $stemp '您的IP地址是:[23.52.1.54]关于nsis中的字符串处理'
${WordFind} "$stemp" "]" "-2{*" $R0
${WordFind} "$R0" "[" "+2*}" $R1
MessageBox MB_OK $R1

Push $stemp
Call GetIP
Pop $R0
MessageBox MB_OK $R0
SectionEnd

Function GetIP
Exch $0
Push $1
Push $2

StrCpy $1 0
IntOp $1 $1 - 1
StrCpy $2 $0 1 $1
StrCmp $2 "]" +2
StrCmp $2 "" 0 -3
StrCpy $0 $0 $1

StrCpy $1 0
IntOp $1 $1 - 1
StrCpy $2 $0 1 $1
StrCmp $2 "[" +2
StrCmp $2 "" 0 -3
IntOp $1 $1 + 1
StrCpy $0 $0 "" $1

Pop $2
Pop $1
Exch $0
FunctionEnd
```
### 9 关于nsis中的API调用
```
#头文件
!include "WordFunc.nsh"
#输出文件
OutFile "关于nsis中的API调用.exe"
#区段
Section "关于nsis中的API调用"
#获得磁盘可用空间
StrCpy $0 "D:\"
System::Call kernel32::GetDiskFreeSpaceEx(tr0,*l,*l,*l.r1)
MessageBox MB_OK "$0的可用空间为:$1"
; Create RECT struct

#获得窗口矩形大小
System::Call `*(i, i, i, i) i .R0`
System::Call `user32::GetClientRect(i $HWNDPARENT, i R0)`
System::Call `*$R0(i.R1, i.R2, i.R3, i.R4) i`
System::Free $R0
MessageBox MB_OK " Left:$R1$\n Top:$R2$\n Right:$R3$\n Bottom:$R4$\n"
SectionEnd
```
### 10 NSIS 常用小问题问答合集


>问： 如何用 NSIS 安装输入法。
>>答： 以下代码：
SetOutPath $SYSDIR
File WBIME.ime
Push "五笔输入法"
Push "$SYSDIR\WBIME.ime"
System::Call "Imm32::ImmInstallIME(t s, t s) i .s"
System::Call "Imm32::ImmIsIME(i s) i .s"
Pop $0
IntCmp $0 1 0 +3 +3
MessageBox MB_OK "输入法安装成功"
Goto +2
MessageBox MB_OK "输入法安装失败"

>问： 如何用NSIS注册字体？
>>答： 以下代码：
!include WinMessages.nsh
Section "MainSection" SEC01
File /oname=$FONTS\tahoma.ttf tahoma.ttf
Push "$FONTS\tahoma.ttf"
System::Call "Gdi32::AddFontResource(t s) i .s"
Pop $0
IntCmp $0 0 0 +2 +2
MessageBox MB_OK "注册字体失败"
SendMessage ${HWND_BROADcast} ${WM_FONTCHANGE} 0 0
SectionEnd

>问： 添加版本号时在脚本中加入下面的代码，则为 NSIS 生成的 exe 添加版本信息。
VIProductVersion "1.2.3.4"
VIAddVersionKey /LANG=${LANG_ENGLISH} "ProductName" "Test Application"
VIAddVersionKey /LANG=${LANG_ENGLISH} "Comments" "A test comment"
VIAddVersionKey /LANG=${LANG_ENGLISH} "CompanyName" "Fake company"
VIAddVersionKey /LANG=${LANG_ENGLISH} "LegalTrademarks" "Test Application is a trademark of Fake company"
VIAddVersionKey /LANG=${LANG_ENGLISH} "LegalCopyright" "?Fake company"
VIAddVersionKey /LANG=${LANG_ENGLISH} "FileDescription" "Test Application"
VIAddVersionKey /LANG=${LANG_ENGLISH} "FileVersion" "1.2.3"
问题就是，能否让属性中语言显示为“中文（中国）”？如附图： 
>>答： 中文 ID 是 2052。 把 ${LANG_ENGLISH} 改为 2052。

>问： 用 2052 之后确实变成“中文（中国）”了。但其他内容仍旧是乱码，不知有什么办法可以解决吗？如附图：
>>答： 版本信息设置语句，放在 !insertmacro MUI_LANGUAGE 的后面，NSIS 要注重次序的。如果使用古典界面，放在 LoadLanguageFile "${NSISDIR}\Contrib\Language files\SimpChinese.nlf" 的后面。 

>问： 在 NSIS 中如何设置工作目录，例如一些文件的快捷方式，还有安装完一个软件后运行一个程序，而这个程序需要检测当前工作目录下的某个文件，这时候设置工作目录尤为重要，否则程序不能正常运行。
>>答：NSIS 中设定工作目录使用 SetOutPath，例如在运行程序的代码 ExecWait "$INSTDIR\test2.exe" 前放入 SetOutPath $INSTDIR，
那么 $INSTDIR 将会成为当前的工作目录，建立快捷方式也会把工作目录设为 $INSTDIR。
　　卸载之前运行某程序只需要把运行指令放到 Function un.onInit 里就行。

>问： NSIS对于安装卸载的ICO图标大小有什么要求?编译的时候出现以下错误提示：
Error finding icon resources: installer, uninstaller icon size mismatch - see the Icon instructions documentation for more information -- failing! 
>>答： 只要保证安装图标与卸载图标大小相同即可。

>问： 在安装的时候不是可以选择多种语言么？但是我怎样实现当选择英文时就装英文版，选择中文时就装中文版？
>>答： 使用以下脚本：
StrCmp $LANGUAGE ${LANG_SIMPCHINESE} 0 +3
File "你需要安装的中文文件"
Goto lbl_finish
File "你需要安装的英文文件"
lbl_finish:

>问： 在NSIS中如何才能做到根据对于注册表键值的判断决定是否写入字串，如果判断出某个key存在，则写入相应的字串，如果不存在，则不写入字串。例如：我先要判断 “ HKLM SOFTWARE\nsis”这个key存不存在。如果存在则写入字串“DispName:nsis”，应该是用“WriteRegStr HKLM "SOFTWARE\nsis" "DispName" "nsis"”。如果不存在这个key，则不写入注册表，继续下面的安装。
>>答： 以下代码实现：
ReadRegStr $0 HKLM SOFTWARE\nsis ""
　IfErrors 0 +2
Goto +2
WriteRegStr HKLM "SOFTWARE\nsis" "DispName" "nsis"

>问：NSIS打包软件安装完毕后选择是否重新启动计算机？
>>答：第一种方法：
SetRebootFlag true
IfRebootFlag 0 +2
同时如果有!define MUI_FINISHPAGE_NOREBOOTSUPPORT 记得删掉。
第二种方法：但其实可以跳出窗口询问
MessageBox MB_YESNO|MB_ICONQUESTION|MB_TOPMOST "请重启以便补丁安装完全及垃圾清理完整!" IDNO +2
Reboot

>问： 如何运行一个安装文件 .inf 
>>答： ExecWait "RunDll32 advpack.dll,LaunchINFSection skins.inf,DefaultInstall"

>问： 能不能在 Section 区段中实现读取INI文件状态来安装
如图所示，若选中单选框1则安装1中定义的文件。若不选中则不安装。若选中单选框2则安装2定义的文件。若不选则不安装。
>>答： 使用以下代码：
!include LogicLib.nsh
Section -post
SetOutPath $INSTDIR
!insertmacro MUI_INSTALLOPTIONS_READ $INI_VALUE "info.ini" "Field 2" "State"
${If} $INI_VALUE = 1
File /a ".\file\fileA.exe"
File /a ".\file\fileB.exe"
${EndIf}
!insertmacro MUI_INSTALLOPTIONS_READ $INI_VALUE "info.ini" "Field 3" "State"
${If} $INI_VALUE = 1
File /a ".\file\fileA.exe"
${EndIf}
SectionEnd
或者使用以下代码：
!include LogicLib.nsh
Section -post
SetOutPath $INSTDIR
!insertmacro MUI_INSTALLOPTIONS_READ $INI_VALUE "info.ini" "Field 2" "State"
${If} $INI_VALUE = 1
;选中时执行的代码
File /a ".\file\fileA.exe"
File /a ".\file\fileB.exe"
${Else}
;不选中时执行的代码
File /a ".\file\fileA.exe"
${EndIf}
SectionEnd

>问： 在安装过程中按“取消”的话，会弹出是否终止安装的确认窗口，请问怎样设置可以让这个窗口不要出现，按“取消”就直接退出呢？
>>答： !define MUI_ABORTWARNING　把这句去掉就可以了....
>问： 如图所示的地方，现在显示的是“setup 将安装...”，除了用自定义字串来修改这个地方以外，如何把这个setup搞成其他的？比如“安装程序现在将...”
>>答： DirText "安装程序将安装 $(^NameDA) 在下列文件夹。要安装到不同文件夹，单击 [浏览(B)] 并选择其他的文件夹。 $_CLICK"
>问： 如何定义欢迎页面的标题字体大小。如下图所示，图三红色框内的标题字体。
>>答： 使用以下脚本
!define MUI_PAGE_CUSTOMFUNCTION_SHOW ChageFONT
!insertmacro MUI_PAGE_WELCOME
Function ChageFONT
　GetDlgItem $0 $MUI_HWND 1201
　createFont $1 "Tahoma" "11" "700"
　SendMessage $0 ${WM_SETFONT} $1 0
FunctionEnd

>问： 关联文件图标后，图标没变化。
>>答： 刷新图标用：
System::Call shell32.dll::SHChangeNotify(l, l, i, i) v (0x08000000, 0, 0, 0)
>问： 我用NSIS做好了一个安装程序，因为数据较多，一共有400多M，用的LZMA压缩方式，做好后的安装程序约200M，但是我发现在运行这个安装程序时会在系统TEMP目录产生一个同安装后的全部内容同样大的临时文件（一边运行一边加大，最后到400多M去了），如果我做的程序小倒没什么，可是这个程序有400多M，除了要写入安装的数据外还要同样大小的空间放临时文件，这样子也实在是太花不来，我想请问：有什么办法能让其在安装时不使用这么多的临时空间吗？安装的脚本是用HM NIS Edit的向导生成的。
>>答： 这是因为 NSIS 在用 LZMA 时采用了固实压缩，何谓固实压缩，其实就是把所有文件统一起来压缩，所以这样压出来的文件更加的小，
同时也带来了一个问题，安装解压的时候，在临时文件夹中生成一个临时文件，随着安装的进程逐渐增大，到最后，
需要临时文件会变成跟原安装程序一样大，也就是说，需要原安装程序 2 倍的空间才可以安装这个程序，所以对于安装大量文件时，这是不适合的。
　　NSIS 2.07 版本之前 LZMA 算法是固实压缩的，没有非固实的选项，如果需要这样做，只有下载非固实压缩的编译器，
但是 2.07 后的 NSIS 的 LZMA 压缩已经改为默认非固实压缩了，所以这个问题同时也不再存在。如果在制作少量文件的安装时，
仍然想取用固实压缩可以加入 /SOLID 参数。像这样： SetCompressor /SOLID lzma

>问： 如何屏蔽如下图中的安装程序校验。
>>答：CRCCheck　on|off|force
指定安装前安装程序是否对自身执行一个 CRC。注意，如果用户使用了 /NCRC 命令行参数，且你没有指定 force 参数时，不会执行 CRC，
这样有可能导致用户安装一个损坏的安装程序。
安装程序 CRC 校验是默认打开的。可以在脚本中用 CRCCheck off 来默认禁止安装程序的 CRC 校验。不过这样做正如解释上说的可能安装会出现问题。
作汉化的最好加上校验，免得安装程序的问题变成你汉化质量的问题。

>问： 记得以前看到有文章介绍过可在NSIS中调用.inf文件安装附加驱动程序，具体实现代码如下形式：
ExecWait "RunDll32 advpack.dll,LaunchINFSection drivers.inf,DefaultInstall"
我在打包一小东东时使用了这一方法，但遇到的问题时，如果在卸载区段里设置能自动卸载安装过的驱动程序呢？
>>答： 能否卸载需要看 INF 文件里面是否有卸载的区段，例如使用 NSIS 卸载 Windows Messenger 可以这样：
ExecWait "RunDll32 advpack.dll,LaunchINFSection $windir\INF\msmsgs.inf,BLC.Remove"
关于 BLC.Remove 的来源，可以打开 msmsgs.inf 文件后，能找到名称为 BLC.Remove 的区段，该区段用于卸载。
区段的名称是编写者自己定制的。不同的inf文件，区段名也可能不同。
其他的 *.inf 文件也可以按照此类做法。

>问：如何让NSIS的安装进度条自动关闭?
>>答：
SetDetailsPrint none
;这里写要隐藏的语句
SetDetailsPrint both
相关知识：
　　4.8.1.33 ShowInstDetails
　　hide|show|nevershow
设置是否显示安装详细信息。你可以设为 hide 来隐藏详细信息但用户可以查看，show 用来默认显示详细信息，
nevershow 可以阻止用户查看任何信息。注意区段里可以使用 SetDetailsView 来更改它的设置。

>问：用 NSIS 做得安装程序怎样能够添加系统服务啊 还有就是怎样在安装了以后服务立刻启动而不用等到重启计算机。
>>答：服务就注册在 HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\ 下，可以用添加注册表项的方式添加。

>问：我想把一小程序用NSIS打包，但用注册机算出来的注册码保存到注册表中时是以Hex数据保存的，能否在安装过程中将输入到Edit框中的数据转换成Hex数据保存到注册表中呢？注册码的形式是“XXXX-XXXX-XXXX-XXXX”！
>>答：试试看这个
!define HKEY_CLASSES_ROOT 0x80000000
!define HKEY_CURRENT_USER 0x80000001
!define HKEY_LOCAL_MACHINE 0x80000002
!define HKEY_USERS 0x80000003
!define HKEY_PERFORMANCE_DATA 0x80000004
!define HKEY_CURRENT_CONFIG 0x80000005
!define HKEY_DYN_DATA 0x80000006
!define ROOT_KEY ${HKEY_CURRENT_USER}
!define SUB_KEY "Software\test"
!define VALUE "KeyValue"
!define DATA "ABC"
System::Call "*(i) i (0) .r0"
System::Call "*(&t1024 '${DATA}') i .r1"
StrLen $2 "${DATA}"
System::Call "Advapi32::RegCreateKey(i ${ROOT_KEY}, t '${SUB_KEY}', i r0) i"
System::Call "*$0(&i8 .r3)"
System::Call "Advapi32::RegSetValueEx(i r3, t '${VALUE}', i 0, i 3, i r1, i r2) i"
System::Call "Advapi32::RegCloseKey(i r0)"
System::Free $1
System::Free $0

>问： 比如，我把 a.exe 用nsis包装好，安装到 c:\helloLib\a.exe，完成后，想把c:\helloLib\添加到 系统环境变量的 path里头，这样，在任何地方输入 a.exe可执行。如何将路径添加到 系统环境变量中？
>>答：以下代码实现：
ReadRegStr $0 HKLM "SYSTEM\CurrentControlSet\Control\Session Manager\Environment" "Path"
WriteRegExpandStr HKLM "SYSTEM\CurrentControlSet\Control\Session Manager\Environment" "Path" "$0;C:\hellolib"
另类方法一： 写注册表，如
[HKEY_CLASSES_ROOT\Applications\a.exe\shell\open\command]
@="yourpath\a.exe"
另类方法二：
[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\App Paths\a.exe]
@="c:\helloLib\a.exe"

>问： 如果是这样写的时候就可以在左边显示页眉位图：
!define MUI_ABORTWARNING
!define MUI_ICON "${NSISDIR}\Contrib\Graphics\Icons\modern-install.ico"
!define MUI_UNICON "${NSISDIR}\Contrib\Graphics\Icons\modern-uninstall.ico"
!define MUI_HEADERIMAGE
!define MUI_HEADERIMAGE_BITMAP "f:\11.bmp"
会显示如附图1。会靠左，但是如果把 MUI_HEADERIMAGE 换成 MUI_HEADERIMAGE_RIGHT 就无法显示位图,只能出现安装图标了，如附图2。
>>答： 把插入的headerimage图片放到右边（默认是在左边）
!define MUI_HEADERIMAGE
!define MUI_HEADERIMAGE_RIGHT
!define MUI_HEADERIMAGE_BITMAP "include\resource\modern-header.bmp"
而不是你所说的修改 !define MUI_HEADERIMAGE，应该是在这句下面添加 !define MUI_HEADERIMAGE_RIGHT

>问： 1. 我做了一个可同时在简繁英下执行的安装程序。 但有个很大的困惑。
刚开始时做的是简体中文版，在简中下当然没问题，但在英/繁下，创建的中文程序组名和写注册表时显示的是乱码。 于是想到写一个三者皆适合的安装程序。
我的做法是这样的，在涉及到创建中文程序组和写注册表时，加入一个判断，如果英文或繁体，则分别创建英文文件名和big5内码文件名。 但在繁体winxp和英文win2k下测试后，发现根本无法正常创建程序组，也无法正常生成uninstall.exe文件。 而写入注册表的中文字符，也不能在繁体系统的注册表中正常显示，而是乱码。
请教高手，这种多语言环境下该怎么处理，才能正常显示呢？ 总不能全部都给创建成英文的吧？
2. 另外有个小问题，如何让nsis做的安装程序不显示具体的安装文件名，而只显示“正在复制文件……”？　象foobar那样，可惜foobar也有个显示详细情况的按钮，我想在复制文件过程中完全不显示和提示被复制的文件情况。 
>>答： 1. 最简单的处理办法就是给需要处理的资源进行一个定义，然后使用 LangString 定义不同的资源。给个示例：
Caption "$(CAPTION)"
LangString CAPTION ${1033} "DreamMail Installation"
LangString CAPTION ${2052} "DreamMail 安装向导"
LangString CAPTION ${1028} "DreamMail 杆翾旧"
2. 可以使用 DetailPrint ，示例：
DetailPrint "正在复制文件..."

 

>问： 如何禁止显示如附图中的 banner 。
>>答： 图示的效果是因为安装程序初始化的时候，也就是 .onInit 函数里需要用到某个文件的时候安装程序需要搜索整个数据区块来把它解压出来，
当安装程序比较大的时候搜索比较费时，这个时候才显示解压百分比。一般都是用 ReserveFile 来避开这种搜索。比如 Function .onInit 里有
InitPluginsDir
File "/oname=$PLUGINSDIR\io.ini" ".\io.ini"
或者其他类似的话，安装程序就需要搜索并解压这个文件
一般在脚本头部加
ReserveFile ".\io.ini"
这样 io.ini 就保存在数据区块的尾部，安装程序初始化的时候就不用搜索整个数据区块了，相当于加快了安装程序的启动速度。
>问： 根据以上方法使用了，确实不会再出现初始化的对话框了，但是在自定义的 InstallOptions 页显示前，程序还是会停顿一段时间，请问这是为何？如何避免？ 
>>答： 某些控件比较消耗时间的，比如显示 ICON、位图 等，如果 InstallOptions 里含有这些控件可能会停顿。
如果 InstallOptions 是第一个页面的话还要把 InstallOptions.dll 加入到 ReserveFile 参数里。
再或者就是进入 InstallOptions 的时候含有比较复杂的指令，比如循环等。
一般在加入　ReserveFile ${NSISDIR}\Plugins\InstallOptions.dll 既可避免。

>问：NSIS打包时运行程序时不显示窗口
>>答：
nsExec::ExecToStack '"$INSTDIR\someprogram.exe" /paramS'
Pop $0 ;;nsExec::ExecToStack 执行结果
Pop $1 ;;someprogram.exe 返回结果
同样有2个插件
ExecCmd plug-in : http://nsis.sourceforge.net/ExecCmd_plug-in
ExecDos plug-in : http://nsis.sourceforge.net/ExecDos_plug-in

>问：NSIS安装包在安装完成后打开主页的代码
>>答：
!define MUI_FINISHPAGE_RUN "$INSTDIR\IMETool.exe"
!define MUI_FINISHPAGE_RUN_TEXT "运行程序"
!define MUI_FINISHPAGE_SHOWREADME
!define MUI_FINISHPAGE_SHOWREADME_FUNCTION Info
!define MUI_FINISHPAGE_SHOWREADME_TEXT "打开主页"
!insertmacro MUI_PAGE_FINISH
Function Info
ExecShell "open" " http://nsis.sf.net/"
Functionend
>问：我想在程序安装完毕时自动运行一个文件,比如批处理文件,网页文件,可执行文件,之类的东东,具体要怎么做?
>>答：Function .onInstSuccess中使用ExecWait ''来调命令
>问：如何根据对于注册表键值的判断决定是否写入字串？
在NSIS中如何才能做到根据对于注册表键值的判断决定是否写入字串，如果判断出某个key存在，则写入相应的字串，如果不存在，则不写入字串。例如：我先要判断“HKLM SOFTWARE\nsis”这个key存不存在。如果存在则写入字串“DispName:nsis”，应该是用“WriteRegStr HKLM "SOFTWARE\nsis" "DispName" "nsis"”。如果不存在这个key，则不写入注册表，继续下面的安装。
>>答：以下代码可以实现
ReadRegStr $0 HKLM SOFTWARE\nsis ""
　IfErrors 0 +2
　Goto +2
WriteRegStr HKLM "SOFTWARE\nsis" "DispName" "nsis"

>问： 组件A 组件B 组件C 均为可选，A可单独安装，B或者C被选择的时候A必须被选择。
>>答： 以下代码
Section "组件 A" aaa
detailprint "A"
SectionEnd
Section "组件 B" bbb
detailprint "B"
SectionEnd
Section "组件 C" ccc
detailprint "C"
SectionEnd
Function .onSelChange
SectionGetFlags ${bbb} $0
SectionGetFlags ${ccc} $1
IntOp $0 $0 & 1
IntOp $1 $1 & 1
IntCmp $0 1 0 +2
　SectionSetFlags ${aaa} 1
IntCmp $1 1 0 +2
　SectionSetFlags ${aaa} 1
FunctionEnd
解释：SectionGetFlags 表示获取某区段的flags状态（就是是否被勾选，选中返回值为1，反之为0）
SectionGetFlags ${bbb} $0 表示获取序号为${bbb}的区段的Flags状态并把返回值输出到变量 $0，C 区段相同。
接着就是 StrCmp ，解释同上。
SectionSetFlags ${aaa} 1 表示设置序号为 ${aaa} 区段的 Flags 状态为 1，即勾选。


>问：当字符串长度超过1024时，怎么办？例如 nsis 里面的语句： RealIniStr $R0 "c:\test.ini" "test" "test"其中 $R0 的长度受 1024 的限制，请问有什么方法可以读取到2000~3000长度的字符串？
>>答：到 http://nsis.sourceforge.net/download/specialbuilds/ 下载 Large strings 版的，可以支持到 8196 字节。

>问：在nsis中如何注册 dll、ocx 等文件？
>>答：nsis 有命令可以注册 dll、ocx 等文件的
regdll "$instdir\xxx.dll"
unregdll "$instdir\xxx.dll" (反注册)
上面2个命令建议在 Section(或卸载的 Section) 中运行。
>问：我在用NSIS打包一小程序时，需要在卸载时解除Mydll.dll文件的注册，因此在Section Uninstall的开始处加入了UnRegDLL "$INSTDIR\Mydll.dll" 但卸载后总是达不到预期的效果，该Dll文件的注册信息不能去掉咯，请问各位大大如何原因啊？
>>答：这样应该可以nsexec::exec "regsvr32 /u $INSTDIR\Mydll.dll"


>问： 怎么让 ＂许可协议＂页面的标题栏，如程序中的“MutliPages 演示”修改为“MutliPages 演示：许可协议”，如附图。
>>答： 首先创建一个函数，如下：
Function LicensePagePre
SendMessage $HWNDPARENT ${WM_SETTEXT} 0 "STR:我爱你"
FunctionEnd
然后在协议页面句子
!insertmacro MUI_PAGE_LICENSE "c:\path\to\licence\YourSoftwareLicence.txt"
之前加入如下语句
!define MUI_PAGE_CUSTOMFUNCTION_PRE LicensePagePre

>问： 如何制作安装包的时候需要调用系统函数来检测当前安装包运行的操作系统的内码页。
>>答： 以下代码显示系统语言：
System::Call "Kernel32::GetSystemDefaultLangID(v ..) i .s"
Pop $0
IntOp $0 $0 & 0xFFFF
MessageBox MB_OK $0

// nsis版本
const char *NSIS_VERSION="v2.46";
// 将标准输出重定向到文件当中
FILE *g_output=stdout;
// 指针的指针**
int main(int argc, char **argv)
/*
#include <stdio.h>
#include <iostream>
using namespace std;
int main(int argc, char** argv)
{
    // 定义一个字符指针数组
    char* arr[] = {"hello","world","nsis"};
    // 定义一个指向指针的指针,arr为数组首地址==&arr[0]
    char **ppi = arr;
    // 输出第2个字符串(ppi+1),从第2个字符开始输出
    cout << *(ppi+1)+1 << endl; // 结果:"orld"
    // 暂停等待输入字符回车
    getchar();    
    return 0;
}
*/
// 显示nsis的版本号
// fprintf格式化数据输出到一个缓冲区或文件
// fflush清除文件缓冲,文件以写方式打开时将缓冲区内容写入文件
  if (argc > 1 && !stricmp(argv[1], "/VERSION"))
  {
    fprintf(g_output,NSIS_VERSION);
    fflush(g_output);
    return 0;
  }
// The /V 开关及后面跟随的 0 ~ 4 数字设定了输出。0=无输出，1=仅错误，2=警告和错误，3=信息、警告和错误，4=全部输出。
  if (argc > 1 && argv[1][0]=='/' && (argv[1][1]=='v' || argv[1][1]=='V'))
  {
    tmpargpos++;
    if (argv[1][2] <= '2' && argv[1][2] >= '0')
    {
      no_logo=1;
    }
  }
// 代码中间功能,设置编译信息,提示信息
// makensis [选项 | script.nsi | - [...]]
/*
选项
/LICENSE 显示一个许可页面。
The /V 开关及后面跟随的 0 ~ 4 数字设定了输出。0=无输出，1=仅错误，2=警告和错误，3=信息、警告和错误，4=全部输出。
The /P 开关及后面跟随的 0 ~ 5 数字设定编译程序进程的优先级。 0=空闲, 1=低于正常, 2=正常 (默认), 3=高于正常, 4=高, 5=立即。
The /O 开关及后面跟随的记录文件告诉编译器输出记录到记录文件而不是屏幕。
/PAUSE 使得 makensis 在退出前暂停，当直接从 Windows 执行时非常有用。
/NOCONFIG 禁止包含 nsisconf.nsh 。没有这个参数的话，安装程序默认从 nsisconf.nsh 读取设置。
/CMDHELP 输出基本的命令用法信息(如果指定了命令)，或所有命令(如果未指定命令)。
/HDRINFO 输出 makensis 编译的选项信息。
/NOCD 禁止把当前目录更改到 .nsi 文件。
使用 /D 开关一次或多次将会把符号添加到全局定义列表 (请看 !define)。
使用 /X 开关一次或多次将会执行你随后指定的代码。例如: "/XAutoCloseWindow false"
对脚本名指定一个破折号(-)将会通知 Makensis 把标准输入作为源来使用。
*/
// 调用build.write_output()进行exe文件写入即生成文件
  if (build.write_output())
  { 
    if (build.display_errors) 
    {
      fprintf(g_output,"Error - aborting creation process\n");
      fflush(g_output);
    }
    return 1;
  }
// 生成主程序 开始输出信息  
  FILE *fp = fopen(build_output_filename,"w+b");
  if (!fp)
  {
    ERROR_MSG("Can't open output file\n");
    return PS_ERROR;
  }





