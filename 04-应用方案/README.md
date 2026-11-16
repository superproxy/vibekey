# 04 应用方案

一份方案 = **key映射程序 × 遥控器型号 × 应用程序**（模板：`axonkey-device-apps`）。

- 文件名：`{产品}.axonkey-{device}`（apps 写在文件内容里）
- 子目录按 key映射程序存放，不构成方案身份

| 路径 | 职责 |
| --- | --- |
| `AxonKey/` | `{产品}.axonkey-{device}` 导入配置 |
| `SayAll/` | 仅当需要虚拟麦另案时（arn9 Win+H 方案不用） |