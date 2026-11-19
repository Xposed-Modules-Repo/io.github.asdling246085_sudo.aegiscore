# AegisCore（护航音轨）

> 本仓库为 LSPosed 官方模块仓库上架版，应用 ID 为 `io.github.asdling246085_sudo.aegiscore`；代码与原 `com.dsh.aegiscore` 系列（1.5.x–7.9.1）完全一致，仅应用 ID 不同。

让微信 / QQ 语音通话时**不把蓝牙耳机切成通话模式**（HFP/SCO），耳机继续留在高清媒体路（A2DP / LHDC / LDAC），收音走**手机机身麦克风**——就是"游戏里开麦、耳机里还是立体声"的那个效果。

- 本仓库发布版包名 `io.github.asdling246085_sudo.aegiscore`（LSPosed 仓库要求 `io.github.<用户名>` 前缀）
- 原包名 `com.dsh.aegiscore` 的老版本继续走自有更新通道；两个包名可共存，功能相同、签名相同
- 只装用户区，不改 `/system` 只读分区
- 需要 Android 11+、LSPosed（Xposed API 82+），Root 方案不限（Magisk / KernelSU + Zygisk Next / 临时 root 均可）
- 主要在 realme GT7 / ColorOS 17 上验证；ColorOS / realme UI / 类 AOSP 机型反馈较好，其他 ROM 需自行确认

## 它做什么

| 层 | 要勾什么 | 管什么 |
|---|---|---|
| 系统层 | 系统框架（`android`） | 拦 App 把整机切进通话模式（`setMode(MODE_IN_COMMUNICATION)`）、拦 `startBluetoothSco` / `setBluetoothScoOn`、拦 `setCommunicationDevice` / `setCommunicationRouteForClient` 把蓝牙钉成通话设备、看门狗把被顶回来的通话模式拉回普通 |
| App 层 | 对应 App（可选） | 录音音源改机身麦、播放用途改媒体、设备表藏掉经典 SCO |

微信 / QQ 的语音引擎走 native，App 层那三项对它们本来就不起作用，勾了只多一分被环境检测的面——**默认只推荐勾「系统框架」+「AegisCore」自己**。

## 主要特性

- **只拦经典 SCO（type=7）**：不误拦 LE Audio 的 `BLE_HEADSET(26)` / `BLE_SPEAKER(27)`，LC3 本来就是双向高清
- **假装成功**：拦下设备类请求时回 `true`，不让 App 以为自己没占到耳机而转头去外放
- **通话模式拉回**：App 把整机切成通话模式后，看门狗把它拉回 `MODE_NORMAL`，媒体流不断
- **收音可选**：强制机身麦克风（`force_mic`）、播放用途改媒体（`voice_to_media`）
- **允许外放开关**：想主动免提就打开，尊重用户选择
- **音量不再跳**：通话中不再做 16 档量化，挂断也不回写媒体音量（7.8 起）
- **真电话放行**：`mode=2` 的真电话一律不干预，避免接电话没声音
- **日志与诊断**：App 内可看拦截计数、设备状态、生成诊断报告

## 安装

1. 安装 APK。
2. LSPosed → 模块 → 启用 **AegisCore**。
3. 作用域只勾两项：`系统框架`、`AegisCore`。
4. 打开 AegisCore → 说明页 → 授予「所有文件访问」（Android 11+ 写日志/配置镜像需要）。
5. 重启手机，让系统框架钩子重新加载。

## 常见问题

**Q：装上没反应？**
A：先看作用域是否勾了「系统框架」；再看 App 内日志有没有「拦住 … 的 SCO 请求」。没有说明钩子没进 system_server，检查 LSPosed 是否正常、模块是否启用、有没有重启。

**Q：微信视频通话还是外放 / 切窄带？**
A：先确认开关（拦通信设备、允许外放）。7.9.1 修了视频路径的外放请求与通信设备列表处理；仍异常请附诊断报告、ROM 版本、微信版本、耳机型号。

**Q：普通电话接不到声音？**
A：真电话已放行；若仍异常，先关模块确认，再反馈。

**Q：和别的音频模块冲突吗？**
A：会。同时装其他改音频路由的模块（音量管理、音效增强、通话录音类）可能互相顶，建议只留一个。

## 说明

- 模块修改系统音频路由，属于侵入性操作，请自行评估；出现异常先关模块再排查。
- 不收集、不上传任何数据；日志只写在本机 `Download/AegisCore/`。
- 更新说明见各 Release 正文，或模块内「检查更新」。
