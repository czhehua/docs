# 使用 PowerShell 修改 Excel 单元格内容

使用 PowerShell 修改单元格可以通过调用 Excel COM 来实现。
- 支持 .xls、.xlsx、.xlsm、.xlsb 等格式。

**注意：**

需要系统中预先安装好 Excel 程序。

## 示例脚本函数：

```powershell
function Replace-ExcelCellText {
    param (
        [Parameter(Mandatory)]
        [string[]]$ExcelPaths,

        [Parameter(Mandatory)]
        [string]$OldText,

        [Parameter(Mandatory)]
        [string]$NewText
    )

    # 创建 Excel COM 对象（只创建一次）
    $excel = New-Object -ComObject Excel.Application
    $excel.Visible = $false   # 不显示 Excel 窗口
    $excel.DisplayAlerts = $false   # 禁止 Excel 弹出任何确认 / 警告对话框
    $excel.AskToUpdateLinks = $false   # 禁止询问是否更新外部链接
    $excel.Calculation = -4135   # 手动计算 xlCalculationManual

    try {
        Write-Host "[Note] Excel files modification will take a few seconds. Please wait..." -ForegroundColor Yellow
        foreach ($path in $ExcelPaths) {
            # 规范路径
            $ExcelPath = $path.Trim()
            if (-not [System.IO.Path]::IsPathRooted($ExcelPath)) {
                $baseDir = if ($PSScriptRoot) { $PSScriptRoot } else { (Get-Location).Path }
                $ExcelPath = Join-Path $baseDir $ExcelPath
            }
            $ExcelPath = [System.IO.Path]::GetFullPath($ExcelPath)

            if (-not [System.IO.File]::Exists($ExcelPath)) {
                throw "Excel file not found: <$ExcelPath>"
            }

            Write-Host "Replace '$OldText' to '$NewText' in $ExcelPath"

            $workbook = $excel.Workbooks.Open($ExcelPath)

            $sheetCount = $workbook.Worksheets.Count
            for ($s = 1; $s -le $sheetCount; $s++) {
                $sheet = $workbook.Worksheets.Item($s)
                $usedRange = $sheet.UsedRange

                if ($null -ne $usedRange) {
                    $rowCount = $usedRange.Rows.Count
                    $colCount = $usedRange.Columns.Count

                    for ($r = 1; $r -le $rowCount; $r++) {
                        for ($c = 1; $c -le $colCount; $c++) {
                            $cell = $usedRange.Item($r, $c)

                            if (
                                -not $cell.HasFormula -and
                                $cell.Value2 -is [string] -and
                                $cell.Value2.Contains($OldText)
                            ) {
                                $cell.Value2 = $cell.Value2.Replace($OldText, $NewText)
                            }

                            [System.Runtime.InteropServices.Marshal]::FinalReleaseComObject($cell) | Out-Null
                            $cell = $null
                        }
                    }
                }

                [System.Runtime.InteropServices.Marshal]::FinalReleaseComObject($usedRange) | Out-Null
                [System.Runtime.InteropServices.Marshal]::FinalReleaseComObject($sheet) | Out-Null
                $usedRange = $null
                $sheet = $null
            }
            # 关闭时自动保存
            $workbook.Close($true)
            [System.Runtime.InteropServices.Marshal]::FinalReleaseComObject($workbook) | Out-Null
            $workbook = $null
        }
    }
    finally {
        Write-Host "[Note] Releasing Excel COM objects. Please wait..." -ForegroundColor Yellow
        if ($excel) {
            $excel.Quit()
            [System.Runtime.InteropServices.Marshal]::FinalReleaseComObject($excel) | Out-Null
        }
        [GC]::Collect()
        [GC]::WaitForPendingFinalizers()
        [GC]::Collect()
    }
}
```

## 使用示例1（单个Excel文件）：

```powershell
Replace-ExcelCellText -ExcelPath "./Resource.xlsx" -OldText "1000" -NewText "1100"
```

## 使用示例2（多个Excel文件）：

```powershell
$ExcelFiles = @(
    "./Glossary.xls"
    "./Glossary.xlsx"
    "./Glossary.xlsm"
    "./Glossary.xlsb"
)
# 将文件名数组指定给 -ExcelPath 参数
Replace-ExcelCellText -ExcelPath $ExcelFiles -OldText "7.0.0.0" -NewText "8.0.0.0"
```
