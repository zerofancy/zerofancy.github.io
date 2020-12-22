title: autoit控制台置顶工具
author: 归零幻想
publishDate: 2019-03-26
editDate: 2019-03-26
tags: [脚本, 控制台, C语言]

<!--config-->

我们学校C语言在线评测系统是bs的，在浏览器中展示要求，文本框中写代码提交评测。

由于本人水平有限，做不到记事本写代码0BUG，所以需要在IDE中写代码然后粘贴到里面。而做题时一边检查代码一边到浏览器里查看要求太费劲了。于是我用autoit写了这个小程序。

置顶所有的C语言控制台窗口，这样调试无疑会方便一些。

用Code Blocks运行的控制台窗口标题会显示文件路径，所以里面肯定带有`.exe`，所以可以利用这一特性：

````autoit
#Region ;**** 参数创建于 ACNWrapper_GUI ****
#AutoIt3Wrapper_icon=C:\Windows\System32\SHELL32.dll|-94
#AutoIt3Wrapper_UseX64=n
#EndRegion ;**** 参数创建于 ACNWrapper_GUI ****
While 1
$var = WinList()

For $i = 1 To $var[0][0]
  ; 只显示带有标题的可见窗口
    If $var[$i][0] <> "" And IsVisible($var[$i][1]) Then
		If StringInStr($var[$i][0],".exe")Then
			WinSetOnTop($var[$i][0],"",1)
		EndIf
        ;MsgBox(0, "详细信息", "标题=" & $var[$i][0] & @LF & "句柄=" & $var[$i][1])
    EndIf
Next
Sleep(100)
WEnd

Func IsVisible($handle)
    If BitAND(WinGetState($handle), 2) Then
        Return 1
    Else
        Return 0
    EndIf

EndFunc   ;==>IsVisible
````

让这个脚本在后台运行，所有控制台窗口直接置顶，然后去浏览器复制测试数据就方便的多。
<!--summary-->
