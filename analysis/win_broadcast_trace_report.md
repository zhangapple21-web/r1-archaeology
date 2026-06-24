# `win_broadcast.ps1` 溯源报告（草案）

> 主题：追踪 R1 `Network_Bridge.ps1` 中 `Broadcast-Presence` 所调用的 `05_LINKS/win_broadcast.ps1` 究竟承担什么职责  
> 状态：基于现有仓库与考古材料的“间接证据版”溯源；**尚未拿到原始 `win_broadcast.ps1` 正文**

## 已知起点

在 R1 片段 `src/05_LINKS/Network_Bridge.ps1` 中，广播相关逻辑非常明确：

```powershell
function Broadcast-Presence {
    Write-Host "   [广播] 向公司内网宣告 R1 主权..." -ForegroundColor Yellow
    $root = "c:\Users\Administrator\Documents\trae_projects\laozhang_ai"
    $broadcastScript = Join-Path $root "05_LINKS\win_broadcast.ps1"
    if (Test-Path $broadcastScript) {
        & $broadcastScript
    }
}
```

这说明 `win_broadcast.ps1` 不是可有可无的小工具，而是 **Network Bridge 在完成本地/局域网穿透配置后，用来做“存在宣告”的第二步动作**。

## 目前能确认的结论

### 1. `win_broadcast.ps1` 属于 `05_LINKS` 域，而不是安全域或执行域

现有 R1 考古材料多次把 `05 LINKS` 定义为：

- `网络桥、广播`
- `通讯系统`
- `最后加载，避免未就绪不对外`

这几个表述拼起来，能确认 `win_broadcast.ps1` 的定位不是“通用脚本”，而是 **对外通信层的一部分**。它应当在基础身份、安全、执行器等域准备完成后才启动。

### 2. 它的主要职责更像“宣告存在/广播主权”，不是“建立隧道”

`Network_Bridge.ps1` 已经把穿透动作单独放在 `Setup-Penetration()` 里，端口口径也写明了：

- `Local Port 3000 (Gateway) <-> LAN Port 8080`

随后才调用 `Broadcast-Presence()`。这意味着：

- **穿透/端口对齐** 是第一层
- **广播宣告** 是第二层

因此 `win_broadcast.ps1` 大概率不是再去做一遍隧道建立，而是把“R1 已上线、可从何处访问、属于谁”这类信息向局域网或公司内网发布。

### 3. “广播”的对象应是内网发现机制，而不是公网代理

当前矿场公网桥的链路是：

`Cloudflare Worker -> zrok reserved share -> OneAPI`

它依靠稳定 URL + 代理转发实现被动访问；而 R1 的桥是：

- 先识别本机局域网身份
- 再建立本地端口/局域网端口对齐
- 再向公司内网宣告主权

两者世界观不同。由此可推断，`win_broadcast.ps1` 的受众更可能是：

- 同网段设备
- 公司内网节点
- 某个局域网发现脚本/轮询器/注册表

而不太像当前这种公网反向代理入口。

## 三种高概率实现形态

在没有原始脚本正文的情况下，从上下文推断，`win_broadcast.ps1` 最可能是以下三类之一：

### 形态 A：UDP / 广播包宣告

可能向某个固定网段或广播地址发送一段简短消息，内容类似：

- R1 节点标识
- 本机 IP
- 服务端口
- 时间戳 / 在线状态

如果是这种实现，它的目标就是让局域网内其它节点“自动发现” R1。

### 形态 B：写入某个共享位置 / 注册位置

也可能不是底层 UDP，而是：

- 写共享目录
- 写某个固定的内网文件
- 调用某个公司内网的登记接口

这类方式本质上也是“广播”，只是实现上是“注册式广播”，不是网络层广播。

### 形态 C：触发另一个存在宣告链条

它可能只是一个入口脚本，内部再调用：

- Windows 网络发现相关命令
- 某个企业内网同步器
- 某个公告/主机登记脚本

从 `Broadcast-Presence` 的命名来看，重点在“presence”，不一定要求它自己完成全部宣告过程。

## 现有证据对这三种形态的支持度

| 推断 | 支持度 | 依据 |
|---|---|---|
| 它负责“存在宣告”而非穿透 | 高 | `Setup-Penetration()` 与 `Broadcast-Presence()` 分离 |
| 它属于通信/广播层 | 高 | `05_LINKS = 网络桥、广播` |
| 它面向内网发现，不面向公网 | 高 | 注释直接写“向公司内网宣告 R1 主权” |
| 它可能发布节点身份 + 访问入口 | 中 | 前一步已拿到本机 IP，且桥脚本强调 `Local Identity` |
| 它可能依赖注册式桥而非 UDP 广播 | 中 | 当前矿场多处基础设施倾向“先桥可用，再补可观测”，说明 R1 也可能采用更稳妥的登记方式 |

## 与当前矿场桥的对照价值

`win_broadcast.ps1` 最值得学的，不一定是它的具体协议，而是它代表的那层设计意识：

1. **桥不仅要可达，还要可被发现**
   - 当前矿场公网桥已经“可达”
   - 但内网、同网段、可信环境里的“自动发现/存在宣告”仍是空白

2. **桥不仅是通道，还是身份宣言**
   - R1 不是简单开放端口，而是用“主权”来描述广播
   - 这意味着它发布的不只是地址，还有“谁拥有这条链路”的身份语义

3. **桥接应区分两层**
   - 第 1 层：连通性（tunnel / port mapping）
   - 第 2 层：可见性（presence / discovery）

当前矿场在第 1 层已经较成熟，但第 2 层还主要依赖人工知道 URL。

## 与现矿场基础设施的映射

在 `mine-seed` 里，当前能找到的桥接证据主要是：

- `07_GUARDIAN/NETWORK_BRIDGE.md`
  - 说明当前桥是 `Cloudflare Worker -> zrok -> OneAPI`
  - Worker 负责桥 key 校验与路径白名单
- `05_TOOLS/bridge-check.sh`
  - 把桥拆成四层验证：Worker、zrok、OneAPI 本地、systemd 服务
- `02_MEMORY/recent_memory/research/shared_bridge_plan.md`
  - 强调“桥先稳，再谈可观测与服务化”

这几份材料能说明：**现矿场的桥关注的是公网可用性与守护；R1 的 `win_broadcast.ps1` 关注的则更像内网存在宣告。**

## 当前缺口

目前还不能确认以下关键问题：

1. `win_broadcast.ps1` 是网络层广播、注册式广播，还是仅仅调用别的登记链？
2. 广播内容具体包含什么字段？
3. 广播后对端如何响应？
4. 广播发现后，访问路径是：
   - 直接连被宣告的端口
   - 先登记再由别的桥接层转发
   - 还是只做“在线存在”提示

## 下一步最优先证据

如果后续能拿到以下任一材料，这篇报告就能从“草案”升级到“确认版”：

1. `05_LINKS/win_broadcast.ps1` 原文
2. 任何调用 `win_broadcast.ps1` 的日志样本
3. 同目录下其他 `05_LINKS/*` 文件
4. 当年公司内网环境里与 R1 发现机制相关的脚本/说明

## 临时结论

在现有证据下，最稳妥的结论是：

> `win_broadcast.ps1` 不是“桥本身”，而是 R1 网络桥的 **存在宣告层**。  
> `Network_Bridge.ps1` 先解决“连得上”，`win_broadcast.ps1` 再解决“别人知道你连上了”。  
> 这正是 R1 与当前矿场公网桥在设计哲学上的核心差异之一：**R1 重发现，现桥重代理。**

