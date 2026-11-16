# AxonKey

命名模板：`axonkey-device-apps`  
落地文件：`{产品}.axonkey-{device}`（apps 在 json 内容）

| 文件 | 方案型号 | 语音 | 本机状态 |
| --- | --- | --- | --- |
| `快意编程v1.axonkey-rc003.json` | rc003 | 未适配（保留） | 存档 |
| `快意编程v1.axonkey-arn9.json` | arn9 | `voice.click` → Win+H | **当前使用** |

说明：AxonKey 对本机实例的 `kind` 仍可能显示 `rc003`（宿主识别）；vibekey **应用方案**按 `arn9` 管理并已写入语音 Win+H。

| 字段 | 含义 |
| --- | --- |
| `device` / `deviceId` | 方案型号 |
| `format` | `axonkey-{device}` |