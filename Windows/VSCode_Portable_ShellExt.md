# 主题：VSCode 便携版 添加右键菜单

**Revision:** 1.0  
**最后更新:** 2025/05/09

## 1. 需求

在无管理员权限的 Windows 系统上使用 VSCode 便携版时，默认不带右键菜单，需要先打开应用后，手动选择打开文件（或文件夹），没有从右键菜单进行操作方便。

为解决此问题，可使用如下的方法手动添加 Shell Extension 支持。

## 3. 添加 Shell Extension 到注册表

创建一个 [**OpenVSCode_HKCU.reg**](reg/OpenVSCode_HKCU.reg) 文件，内容如下，注意其中的文件路径（分隔符需使用双反斜杠）：

```txt
Windows Registry Editor Version 5.00

[HKEY_CURRENT_USER\SOFTWARE\Classes\*\shell\VSCode]
@="Open with VSCode"
"Icon"="C:\\Tools\\VSCode\\Code.exe"

[HKEY_CURRENT_USER\SOFTWARE\Classes\*\shell\VSCode\command]
@="\"C:\\Tools\\VSCode\\Code.exe\" \"%1\""

[HKEY_CURRENT_USER\SOFTWARE\Classes\Directory\shell\VSCode]
@="Open with VSCode"
"Icon"="C:\\Tools\\VSCode\\Code.exe"

[HKEY_CURRENT_USER\SOFTWARE\Classes\Directory\shell\VSCode\command]
@="\"C:\\Tools\\VSCode\\Code.exe\" \"%1\""

```

导入后，选中文件或文件夹右击，可以看到 `Open with VSCode` 菜单。

（完）
