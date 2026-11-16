---
name: vibekey-input-device
description: >-
  Builds vibekey layers 01-04 and AxonKey application schemes
  (axonkey-device-apps). Use when adding a remote model (rc003, arn9), creating
  {产品}.axonkey-{device}.json, mapping voice when BLE mic is unsupported
  (Win+H toggle), or updating this skill / 应用方案.
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

## 检查

- [ ] 型号只用 `deviceId`，不写「设备规范」
- [ ] 文件名 `{产品}.axonkey-{device}`
- [ ] arn9：`voice.click` = Win+H；longPress 空
- [ ] arn9：不写 SayAll 依赖
- [ ] 其它键与约定型号一致（当前=rc003）