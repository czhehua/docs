# 使用 PowerShell 替换文本

在 Linux 等环境中可以通过 sed 进行文本替换。而在 Windows 环境中，使用 PowerShell 脚本也可以实现文本替换功能。

## 示例脚本函数：

函数支持 UTF8、UTF16-LE、ANSI等编码文件的字符串替换。

```powershell
function Replace-Text {
    param (
        [Parameter(Mandatory = $true)]
        [string[]]$FileName,

        [ValidateSet("ANSI","UTF8","Unicode","ShiftJIS","GB2312")]
        [string]$Encoding = "UTF8",

        [Parameter(Mandatory = $true)]
        [string]$OldString,

        [Parameter(Mandatory = $true)]
        [string]$NewString
    )

    # 根据 Encoding 参数转换成 .NET Encoding 对象
    switch ($Encoding) {
        "ANSI"      { $enc = "Default" }
        "UTF8"      { $enc = "UTF8" }
        "Unicode"   { $enc = "Unicode" }
        "ShiftJIS"  { $enc = [System.Text.Encoding]::GetEncoding(932) }
        "GB2312"    { $enc = [System.Text.Encoding]::GetEncoding(936) }
    }

    foreach ($file in $FileName) {

        # 处理相对路径，获取绝对路径
        try {
            $resolvedPath = Resolve-Path -Path $file -ErrorAction Stop
            $absFilePath = $resolvedPath.Path
        } catch {
            throw "File not found: $file"
        }

        # 使用 .NET 判断文件是否存在（双保险）
        if (-not [System.IO.File]::Exists($absFilePath)) {
            throw "File not found: $absFilePath"
        }

        # 输出信息
        Write-Host "Replace '$OldString' to '$NewString' in $absFilePath ($Encoding)"

        # 读取文件
        $content = Get-Content -Path $absFilePath -Encoding $enc

        # 替换字符串（按字面值）
        $content = $content -replace [regex]::Escape($OldString), $NewString

        # 写回文件（覆盖原文件）
        Set-Content -Path $absFilePath -Encoding $enc -Value $content
    }
}
```

## 使用示例1（替换 UTF8 文本文件）：

```powershell
# -Encoding UTF8 参数为默认值，可省略
Replace-Text -FileName "./Application.cpp" -OldString "1000" -NewString "1100"
```

## 使用示例2（替换 UTF16-LE .RC文件）：

```powershell
# 必须指定 -Encoding Unicode 参数
Replace-Text -FileName "./Application.rc" -Encoding Unicode -OldString "1.0.0.0" -NewString "1.1.0.0"
Replace-Text -FileName "./Application.rc" -Encoding Unicode -OldString "1,0,0,0" -NewString "1,1,0,0"
```

## 使用示例3（替换 GB2312 或 ShiftJIS 文件）：

```powershell
# 必须指定 -Encoding 参数
Replace-Text -FileName "./GB2312.txt" -Encoding GB2312 -OldString "黑白" -NewString "彩色"
Replace-Text -FileName "./ShiftJIS.txt" -Encoding ShiftJIS -OldString "あいうえお" -NewString "かきくけこ"
```

## 使用示例4（批量替换多个文本文件）：

```powershell
$TextFiles = @(
    "./Test1.txt"
    "./Test2.log"
    "./Test3.csv"
)
# 将文件名数组指定给 -FileName 参数
Replace-Text -FileName $TextFiles -OldString "ABCD" -NewString "EFGH"
```
