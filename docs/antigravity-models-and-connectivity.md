# Antigravity 模型与连接排查

## 模型目录与档位

插件通过 `fetchAvailableModels` 获取当前账号的模型目录，dsh 请求模型列表时会重新查询；请求失败时使用内置备用列表。Antigravity 的目录可能只返回 `gemini-3.8-flash-tiered` 或 `gemini-3.7-flash-tiered`，而客户端需要展示 High、Medium、Low 三个档位。插件从这两个目录项生成对应的选择项；调用时仍向上游发送 `*-tiered` 模型 ID，并在 `generationConfig.thinkingConfig.thinkingLevel` 中传入所选档位。不要把 `*-flash-high` 等选择项直接作为上游 `model` 发送。

相关代码：`lib/code-assist.js` 的 `fetchAvailableModels`、`antigravityTieredAlias`，以及 `lib/vendors/antigravity.js` 的 `streamOnce`。上游目录可能随账号和时间变化，备用列表不能代替实时目录。

## 默认连接与账号代理

账号槽位未设置 `proxyUrl` 时，插件使用 Node 的默认 `fetch`。这表示插件不指定代理；操作系统仍可通过 TUN 路由接管请求。`singbox_tun` 存在并不表示插件需要绑定 TUN 地址；先用系统路由查询确认目标地址是否已经进入 TUN。强制绑定 TUN 地址不能修复 TUN 后续链路的连接超时。

某些 v2rayN TUN 配置会由 sing-box 接管系统流量，再转给本机的 SOCKS 入口，由 xray 继续处理。因此在 TUN 模式下看到本地 SOCKS 端口监听是正常的。具体端口与链路以当前机器的只读配置为准，不要把本机地址或凭据提交到仓库。

如果需要让**某个 dsh 账号**使用现有本地 SOCKS 入口，在本机 Web profile 的 `cordis.patch.yml` 中，为现有 `slots` 数组内的该账号增加 `proxyUrl: "socks5://localhost:<port>"`，并保留其余账号槽位。此配置只影响该账号，不会修改全局代理、TUN 或系统路由。若插件设置接口返回 `settings not ready`，使用本机 profile 配置层；重启 dsh 后从插件配置接口确认该槽位的 `proxyUrl` 已生效。

账号路由代码：`lib/accounts.js` 的 `normalizeSlots` 保留 `proxyUrl`；`lib/index.js` 的 `fetchForRef` 选取该账号的连接方式；`lib/adapter.js` 与账号检查、模型目录、用量请求共用该连接方式；`lib/proxy.js` 实现 SOCKS/HTTP 连接。修改插件仓库源码不会自动更新本机已安装副本。

## `fetch failed` 的判断顺序

1. 查看错误底层原因。`ETIMEDOUT`、`UND_ERR_CONNECT_TIMEOUT` 属于建立连接失败；先不要归因为模型 ID、思考档位或配额。HTTP 404、429 表示请求已经到达上游，应分别检查模型映射或限额。
2. 分别测试默认连接与该账号配置的连接方式，并测试实际生成请求。仅凭模型列表或连接探针成功，不能证明流式生成可用。
3. 检查系统路由与 TUN 状态时只做只读操作。未经用户明确授权，不重启或修改代理进程、TUN、系统路由和全局代理配置。

`streamOnce` 会对尚未输出内容的 `TypeError` 等临时错误最多尝试两次；重试可能延长等待，但不会让原本可建立的连接变成连接超时。使用改动前的插件代码也复现连接超时时，应继续检查当前连接路径，而不是仅撤回模型档位映射。
