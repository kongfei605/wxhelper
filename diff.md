# WeChatFerry 与 wxhelper 功能差异分析

通过对比 WeChatFerry (WCF) 和 wxhelper 的源码，主要针对用户提到的“发送 XML 和富文本信息”功能，得出以下差异分析：

| API endpoint / Feature | WeChatFerry | wxhelper | Status |
| :--- | :--- | :--- | :--- |
| **发送文本消息** | `send_text` | `/api/sendTextMsg` | Both support |
| **发送图片消息** | `send_image` | `/api/sendImagesMsg` | Both support |
| **发送文件消息** | `send_file` | `/api/sendFileMsg` | Both support |
| **发送 XML 消息** | `send_xml` | `/api/sendXmlMsg` | wxhelper just added |
| **发送富文本消息** | `send_rich_text` | `/api/forwardPublicMsg` | Both support |
| **信息防撤回** | *N/A (Broken)* | `/api/antiRevoke` | wxhelper exclusive feature |

## 1. 富文本信息 (Rich Text)
*   **WeChatFerry**: 提供了 `send_rich_text` 接口，接收参数包括 `receiver`, `title`, `url`, `thumbUrl`, `digest`, `account`, `name`。
*   **wxhelper**: **已经实现并开放了完全相同的功能**。在 wxhelper 中，该功能被称为 `ForwardPublicMsg`，对应的 HTTP 接口为 `/api/forwardPublicMsg`。它的底层逻辑是构建一个 `MMReaderItem`，然后通过 `kForwordPublicMsg` (偏移 `0xddc6c0`) 发送出去。
*   **结论**: wxhelper 已经具备此功能，无需重复开发。

## 2. 发送 XML 消息 (XML Msg)
*   **WeChatFerry**: 提供了 `send_xml` 接口，允许直接传入任意 XML 字符串、接收者 wxid 和 XML type (如 33, 36) 来发送自定义 XML 消息。底层在 x86 环境下通过四个汇编 Call 实现。
*   **wxhelper**: **已实现该功能**。在 `Manager` 中补充了 `SendXmlMsg` 函数，并暴露了 HTTP 接口 `/api/sendXmlMsg`，支持通过传入 XML 字符串、wxid 和消息类型来发送自定义内容，补齐了之前的缺失。
*   **结论**: 功能已上线。

## 3. 信息防撤回 (Anti-Revoke)
*   **WeChatFerry**: 内部虽然预留了一个 `revoke_message` 函数，但其本身目的是试图“主动撤回自己的消息”，且处于 `#if 0` 废弃状态。并没有拦截别人撤回消息的功能。
*   **wxhelper**: **已实现该功能，且为独有特性**。通过 Detours Hook 技术拦截了底层的 `ChatRevokeMgr::revokeMsg`。对外提供 `/api/antiRevoke` 控制开关，在开启状态下，当收到对方撤回指令时，直接返回成功，阻止底层消息从本地数据库和界面移除。
*   **结论**: wxhelper 在此功能上实现了对 WCF 的反超。
