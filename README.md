# winget-reinstall PowerShell Proxy Function

## 説明

PowerShellプロファイルに設定することで`winget`に`reinstall`サブコマンドを追加するプロキシ関数。

`winget reinstall <package>`で指定したパッケージをアンインストール後、再インストールする。パッケージが存在しない場合はインストールのみ実行する。

## セットアップ

1. PowerShellで以下のコマンドを実行し、プロファイルファイルを開く。

    ```powershell
    code $PROFILE
    ```

    ファイルが存在しない場合は`New-Item -Path $PROFILE -ItemType File -Force`を実行後に再度試すこと。

2. 開いたファイルに下記の[コード](#コード)を貼り付け、保存する。

3. PowerShellを再起動するか、以下を実行してプロファイルを再読み込みする。

    ```powershell
    . $PROFILE
    ```

## 使用法

```powershell
# 単一パッケージ
winget reinstall "Google Chrome"

# 複数パッケージ
winget reinstall "PowerToys" "7-Zip"

# reinstall以外のコマンドは通常通り動作する
winget upgrade --all
```

---

## コード

```powershell
function winget {
    [CmdletBinding()]
    param(
        [Parameter(ValueFromRemainingArguments = $true)]
        [string[]]$Arguments
    )

    $wingetExePath = (Get-Command winget.exe -ErrorAction SilentlyContinue).Source
    if (-not $wingetExePath) {
        Write-Error "winget.exe が見つかりませんでした。PATHを確認してください。"
        return
    }

    if ($Arguments.Count -gt 0 -and $Arguments[0].Equals('reinstall', 'InvariantCultureIgnoreCase')) {

        if ($Arguments.Count -gt 1) {
            $packageNames = $Arguments[1..($Arguments.Count - 1)]

            foreach ($pkg in $packageNames) {
                & $wingetExePath uninstall --name $pkg --source winget --accept-source-agreements
                & $wingetExePath install --name $pkg --source winget --accept-source-agreements --accept-package-agreements
            }
        }
        else {
            Write-Error "パッケージ名を指定してください。例: winget reinstall <PackageName>"
        }
    }
    else {
        & $wingetExePath @Arguments
    }
}
```
