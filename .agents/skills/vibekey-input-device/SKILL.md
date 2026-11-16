---
name: vibekey-input-device
description: >-
  Builds vibekey layers 01-04 and AxonKey application schemes
  (axonkey-device-apps). Use when adding a remote model (rc003, arn9), creating
  {产品}.axonkey-{device}.json, mapping voice when BLE mic is unsupported
  (Win+H toggle), syncing mappings via axonkey-cli, or updating this skill /
  应用方案.
---

# vibekey 输入设备 / 应用方案

## 概念

| 词 | 含义 |
| --- | --- |
| `device` / `deviceId` | **型号** |
| 应用方案 | key映射程序 × 遥控器型号 × 应用程序 |
| 命名模板 | `axonkey-device-apps` |
| 落地文件 | `{产品}.axonkey-{device}`；apps 在内容 |
| BLE 录音不可用 | 不走虚拟麦；`voice.click` → `Win+H`；再按再发 Win+H 结束 |

## 目录

```
01-遥控器规范/{型号}.md|.json
02-程序能力/{App}.md
03-关联/{型号}/…
04-应用方案/AxonKey/{产品}.axonkey-{型号}.json
```

## 现成方案

| 方案 | 文件 | 语音 |
| --- | --- | --- |
| axonkey-rc003-apps | `快意编程v1.axonkey-rc003.json` | 未适配 / 保留 |
| axonkey-arn9-apps | `快意编程v1.axonkey-arn9.json` | Win+H 开关录音（**当前使用**；宿主 kind 或仍为 rc003） |

## 原则

1. 一文件一职责；表格式。
2. 按键以实物为准。
3. 映射程序不支持蓝牙录音 → AxonKey 内 `shortcut` `["Win","H"]`，仅 `click`。
4. 落地名 `{产品}.axonkey-{device}`。
5. arn9 不依赖 SayAll。
6. 改键后必须用 `axonkey-cli` 写入本机实例，不能只改仓库 JSON。

## 落地 arn9（Win+H）

1. `01/arn9`：公共键同 rc003；`voice.click` = Win+H。
2. `03/arn9`：关联注明语音→Win+H。
3. `04/AxonKey/快意编程v1.axonkey-arn9.json`：各 profile 的 `voice.click` 写入 Win+H；其它键复制 rc003。
4. `format`=`axonkey-arn9`，`deviceId`=`arn9`。
5. 导入 AxonKey 后点语音键应打开系统听写；再点关闭/完成。

### Win+H 行为片段

```json
{
  "id": "voice-click-win-h",
  "enabled": true,
  "type": "shortcut",
  "keys": ["Win", "H"]
}
```

## Cursor：确认长按提交

前提：Cursor Settings → Agents → 开启 **Submit with Ctrl+Enter**（Enter=换行）。

| 键 | 触发 | 动作 |
| --- | --- | --- |
| 确认 | 单击 | Enter（换行） |
| 确认 | 长按 | Mod+Enter（提交；Win 上 Mod=Ctrl） |

## axonkey-cli（本机同步）

CLI：`%LOCALAPPDATA%\Axonkey\axonkey-cli.exe`  
本机实例（当前）：`18e7d7532e470190-0`（kind 可能显示 `rc003`，方案仍按 arn9）  
PowerShell 会剥 JSON 引号，**必须**用 `ProcessStartInfo` 传 `--sequence`。

### 查询

```powershell
$cli = "$env:LOCALAPPDATA\Axonkey\axonkey-cli.exe"
& $cli status --json
& $cli device list --json
& $cli profile list --instance 18e7d7532e470190-0
& $cli profile application-list --instance 18e7d7532e470190-0
& $cli mapping list --instance 18e7d7532e470190-0 --profile "快意编程v1-Cursor" --json
```

### 创建 profile + 绑应用

```powershell
$cli = "$env:LOCALAPPDATA\Axonkey\axonkey-cli.exe"
$inst = "18e7d7532e470190-0"
& $cli profile create --instance $inst --name "快意编程v1-Cursor"
& $cli profile application-set --instance $inst `
  --application "windows:c:\users\59300\appdata\local\programs\cursor\cursor.exe" `
  --profile "快意编程v1-Cursor"
```

创建后把返回的 `profile.id` 写回 `04/.../快意编程v1.axonkey-arn9.json`（前缀 `arn9:`）。

### 写映射（含 Cursor 确认长按）

```powershell
$cli = "$env:LOCALAPPDATA\Axonkey\axonkey-cli.exe"
$inst = "18e7d7532e470190-0"
$profile = "快意编程v1-Cursor"

function Invoke-Axon([string]$Arguments) {
  $psi = New-Object System.Diagnostics.ProcessStartInfo
  $psi.FileName = $cli
  $psi.Arguments = $Arguments
  $psi.UseShellExecute = $false
  $psi.RedirectStandardOutput = $true
  $psi.RedirectStandardError = $true
  $psi.CreateNoWindow = $true
  $p = [System.Diagnostics.Process]::Start($psi)
  $out = $p.StandardOutput.ReadToEnd()
  $err = $p.StandardError.ReadToEnd()
  $p.WaitForExit()
  if ($p.ExitCode -ne 0) { throw "FAIL($($p.ExitCode)): $Arguments`n$err$out" }
  return $out
}
function Esc-Json([string]$json) {
  return '"' + ($json -replace '\\','\\' -replace '"','\"') + '"'
}
function Set-Seq([string]$button, [string]$trigger, [string]$json) {
  $esc = Esc-Json $json
  Invoke-Axon "mapping set --instance $inst --profile `"$profile`" --button $button --trigger $trigger --sequence $esc" | Out-Null
}

Set-Seq voice click '[{"type":"shortcut","keys":["Win","H"],"enabled":true}]'
Set-Seq up click '[{"type":"wheel","direction":"up","enabled":true}]'
Set-Seq down click '[{"type":"wheel","direction":"down","enabled":true}]'
Set-Seq back click '[{"type":"key","key":"Esc","enabled":true}]'
Set-Seq home click '[{"type":"shortcut","keys":["Alt","Tab"],"enabled":true}]'
Set-Seq confirm click '[{"type":"key","key":"Enter","enabled":true}]'
Set-Seq confirm longPress '[{"type":"shortcut","keys":["Mod","Enter"],"enabled":true}]'
Set-Seq left click '[{"type":"shortcut","keys":["Mod","PageUp"],"enabled":true}]'
Set-Seq right click '[{"type":"shortcut","keys":["Mod","PageDown"],"enabled":true}]'
Set-Seq menu click '[{"type":"shortcut","keys":["Mod","P"],"enabled":true}]'
```

单条等价命令（确认长按提交）：

```text
axonkey-cli mapping set --instance 18e7d7532e470190-0 --profile "快意编程v1-Cursor" --button confirm --trigger longPress --sequence "[{\"type\":\"shortcut\",\"keys\":[\"Mod\",\"Enter\"],\"enabled\":true}]"
```

### 回读核对

```powershell
& $cli mapping list --instance 18e7d7532e470190-0 --profile "快意编程v1-Cursor" --json
# 期望含：confirm/click → Enter；confirm/longPress → Mod+Enter
```

## 检查

- [ ] 型号只用 `deviceId`，不写「设备规范」
- [ ] 文件名 `{产品}.axonkey-{device}`
- [ ] arn9：`voice.click` = Win+H；voice.longPress 空
- [ ] arn9：不写 SayAll 依赖
- [ ] 其它键与约定型号一致（当前=rc003）
- [ ] Cursor：`confirm.longPress` = Mod+Enter；已 `axonkey-cli` 写入并回读
- [ ] 改键后已更新 `agents/sync/sync-log.md`
