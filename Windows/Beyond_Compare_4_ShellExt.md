# 主题：Beyond Compare 4 便携版 添加右键菜单

**Revision:** 1.0  
**最后更新:** 2025/04/29

**注意:** 

- 使用 Beyond Compare 4 前，请先确认你已经从 IT 部门申请并获得了授权。  
- Beyond Compare 4 按用户数量授权，已被授权的用户可以安装在多台设备上。

## 1. 需求

在无管理员权限的 Windows 系统上使用 Beyond Compare 4 便携版时，默认不带右键菜单，需要先打开应用后，手动选择比较文件（或文件夹），没有从右键菜单进行操作方便。

为解决此问题，可使用如下的方法手动添加 Shell Extension 支持。

## 2. 从安装版中复制所需的 dll 文件

便携版默认不带 Shell Extension 所需的 dll 文件，可以从安装版复制：

例如，从 "C:\Program Files\Beyond Compare 4" 文件夹中找到如下文件，复制到便携版目录：

- 32位：BCShellEx.dll
- 64位：BCShellEx64.dll

## 3. 添加 Shell Extension 到注册表

创建一个 [**bc4_portable_user_shell_extension.reg**](reg/bc4_portable_user_shell_extension.reg) 文件，内容如下，注意其中的 dll 路径（分隔符需使用双反斜杠）：

```txt
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\SOFTWARE\Classes\CLSID\{57FA2D12-D22D-490A-805A-5CB48E84F12A}]
@="CirrusShellEx"


; Modify the 64-bit BCShellEx64.dll path here (use double backslash)

[HKEY_CURRENT_USER\SOFTWARE\Classes\CLSID\{57FA2D12-D22D-490A-805A-5CB48E84F12A}\InProcServer32]
@="C:\\Tools\\BCompare\\BCShellEx64.dll"
"ThreadingModel"="Apartment"

[HKEY_CURRENT_USER\SOFTWARE\Wow6432Node\Classes\CLSID\{57FA2D12-D22D-490A-805A-5CB48E84F12A}]
@="CirrusShellEx"

; Modify the 32-bit BCShellEx.dll path here (use double backslash)

[HKEY_CURRENT_USER\SOFTWARE\Wow6432Node\Classes\CLSID\{57FA2D12-D22D-490A-805A-5CB48E84F12A}\InProcServer32]
@="C:\\Tools\\BCompare\\BCShellEx.dll"
"ThreadingModel"="Apartment"

[HKEY_CURRENT_USER\SOFTWARE\Classes\*\shellex\ContextMenuHandlers\CirrusShellEx]
@="{57FA2D12-D22D-490A-805A-5CB48E84F12A}"

[HKEY_CURRENT_USER\SOFTWARE\Classes\Directory\shellex\ContextMenuHandlers\CirrusShellEx]
@="{57FA2D12-D22D-490A-805A-5CB48E84F12A}"

[HKEY_CURRENT_USER\SOFTWARE\Classes\Folder\shellex\ContextMenuHandlers\CirrusShellEx]
@="{57FA2D12-D22D-490A-805A-5CB48E84F12A}"

[HKEY_CURRENT_USER\SOFTWARE\Classes\lnkfile\shellex\ContextMenuHandlers\CirrusShellEx]
@="{57FA2D12-D22D-490A-805A-5CB48E84F12A}"

[HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Shell Extensions\Approved]
"{57FA2D12-D22D-490A-805A-5CB48E84F12A}"="Beyond Compare 4 Shell Extension"

```

双击文件运行导入注册表，注销用户后再次登录确认右键菜单已生效。

※以上的注册表是为 Current User 添加右键菜单，无需管理员权限。如果需要为 All Users 添加，将 "HKEY_CURRENT_USER" 改为 "HKEY_LOCAL_MACHINE"

## 4. 删除右键菜单

如删除了Beyond Compare 4 便携版，可使用以下 [**bc4_remove_shell_extension.reg**](reg/bc4_remove_shell_extension.reg) 文件，删除右键菜单：

```
Windows Registry Editor Version 5.00

; Remove settings in BC4 key

[-HKEY_CURRENT_USER\Software\Scooter Software\Beyond Compare 4\BcShellEx]


; Remove keys under HKEY_CURRENT_USER (current user install)

[-HKEY_CURRENT_USER\Software\Classes\CLSID\{57FA2D12-D22D-490A-805A-5CB48E84F12A}]
[-HKEY_CURRENT_USER\Software\Wow6432Node\Classes\CLSID\{57FA2D12-D22D-490A-805A-5CB48E84F12A}]

[-HKEY_CURRENT_USER\Software\Classes\*\shellex\ContextMenuHandlers\CirrusShellEx]
[-HKEY_CURRENT_USER\Software\Classes\Directory\shellex\ContextMenuHandlers\CirrusShellEx]
[-HKEY_CURRENT_USER\Software\Classes\Folder\shellex\ContextMenuHandlers\CirrusShellEx]
[-HKEY_CURRENT_USER\Software\Classes\lnkfile\shellex\ContextMenuHandlers\CirrusShellEx]

[HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Shell Extensions\Approved]
"{57FA2D12-D22D-490A-805A-5CB48E84F12A}"=-


; Remove keys under HKEY_LOCAL_MACHINE (install for all users)

[-HKEY_LOCAL_MACHINE\Software\Classes\CLSID\{57FA2D12-D22D-490A-805A-5CB48E84F12A}]
[-HKEY_LOCAL_MACHINE\Software\Wow6432Node\Classes\CLSID\{57FA2D12-D22D-490A-805A-5CB48E84F12A}]

[-HKEY_LOCAL_MACHINE\Software\Classes\*\shellex\ContextMenuHandlers\CirrusShellEx]
[-HKEY_LOCAL_MACHINE\Software\Classes\Directory\shellex\ContextMenuHandlers\CirrusShellEx]
[-HKEY_LOCAL_MACHINE\Software\Classes\Folder\shellex\ContextMenuHandlers\CirrusShellEx]
[-HKEY_LOCAL_MACHINE\Software\Classes\lnkfile\shellex\ContextMenuHandlers\CirrusShellEx]

[HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Shell Extensions\Approved]
"{57FA2D12-D22D-490A-805A-5CB48E84F12A}"=-

```

※以上的注册表是为 Current User 以及 All Users 添加的右键菜单均删除。若当前用户无需管理员权限，可将后半段的 "HKEY_LOCAL_MACHINE" 的部分删除后再导入 .reg 文件。

（完）