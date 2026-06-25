# R1 `win_broadcast.ps1` 溯源与架构重建：广播宣告机制考古

> **前置说明**：`win_broadcast.ps1` 源码碎片未被固化到当前仓库（github.com/zhangapple21-web/r1-archaeology 中不存在该文件）。以下分析基于：
> 1. `Network_Bridge.ps1` 中对它的引用与调用方式
> 2. R1 系统设计哲学（多宇宙、主权宣告、内网优先）
> 3. Windows 原生广播/发现协议的合理推断
> 4. PowerShell 生态中可用的广播/多播实现模式

## 已知线索（从 Network_Bridge.ps1 提取）

```powershell
# Network_Bridge.ps1 中的 Broadcast-Presence 函数
function Broadcast-Presence {
    Write-Host "   [广播] 向公司内网宣告 R1 主权..." -ForegroundColor Yellow
    $root = "c:\\Users\\Administrator\\Documents\\trae_projects\\laozhang_ai"
    $broadcastScript = Join-Path $root "05_LINKS\\win_broadcast.ps1"
    if (Test-Path $broadcastScript) {
        & $broadcastScript
    }
}
```

**关键特征**：
1. **调用方式**：使用 `&` 运算符调用（非点源 `.` 调用），说明 broadcast 脚本在独立作用域运行，不会污染调用者变量
2. **路径签名**：`05_LINKS\win_broadcast.ps1` 与 Network_Bridge.ps1 同属 `05_LINKS` 目录，按 R1 目录语义，`05_LINKS` 是"链接/网络"域
3. **无参数传递**：调用时不传参，说明 broadcast 脚本内有自主发现机制（读取本机配置、自动确定广播目标）
4. **健康检查**：有 `Test-Path` 检查，说明 broadcast 脚本是可选组件——桥可以无广播运行
5. **主权叙事**："宣告 R1 主权"——不是"发现服务"，而是"宣告存在"

## 广播机制重建（基于 Windows + PowerShell 技术栈）

### 候选协议分析

| 协议 | 端口 | 特点 | 与 R1 气质的匹配度 |
|------|------|------|-------------------|
| **UDP 广播** (255.255.255.255) | 自定义 | 简单、子网内全可达、无ACK保证 | ⭐⭐⭐⭐⭐ 最可能的选择 |
| **mDNS** (Multicast DNS) | 5353 | 零配置、Apple Bonjour 体系、Linux 友好 | ⭐⭐⭐ Windows 原生无内置，需另装 |
| **LLMNR** (Link-Local Multicast Name Resolution) | 5355 | Windows 原生、链路本地多播 | ⭐⭐⭐ 但主要限名称解析，不适合通用宣告 |
| **UDP 组播** (Multicast) | 自定义 | 需 IGMP、可控范围 | ⭐⭐⭐⭐ 但有管理开销 |
| **WSD** (Web Services Dynamic Discovery) | 3702 | Windows 原生、SOAP/WS-Discovery | ⭐⭐ 太重，不符 R1 精简风格 |
| **SSDP** (Simple Service Discovery Protocol) | 1900 | UPnP 基础、HTTPU 格式 | ⭐⭐ 同样偏重 |

**最可能的实现：UDP 广播（自定义协议格式）**

理由：
1. R1 整体设计偏向"简洁、直接、可复盘"——不用额外框架
2. `05_LINKS` 目录语义就是"链接"，广播是最直接的宣告手段
3. 没有依赖外部库/服务的痕迹（整座仓库都是原生脚本）
4. UDP 广播天然适合内网发现，且 PowerShell 内置 `[System.Net.Sockets.UdpClient]` 原生支持

### 重建的典型 `win_broadcast.ps1` 架构

```powershell
# R1 LOCK SYSTEM - Broadcast Presence Module
# 目标：在内网宣告 R1 节点存在，供同网段其他节点发现

param(
    [int]$BroadcastPort = 45911,      # R1 保留端口（非标准端口，减小冲突）
    [string]$BroadcastAddress = "255.255.255.255"
)

# 1. 身份声明构建
function Build-PresencePayload {
    $hostname = [System.Net.Dns]::GetHostName()
    $localIP = (Get-NetIPAddress -AddressFamily IPv4 |
        Where-Object { $_.InterfaceAlias -like "*Wi-Fi*" -or $_.InterfaceAlias -like "*Ethernet*" } |
        Select-Object -First 1).IPAddress
    
    # 宇宙标识（从 KERNEL_MOUNT 的安全配置读取）
    $universeTag = "WIN"  # 当前运行宇宙
    
    # 可用服务声明
    $services = @{
        gateway = "3000"      # 来自 Network_Bridge.ps1 的 Gateway 端口
        bridge  = "8080"      # LAN 端口
    }
    
    return @{
        type     = "R1_PRESENCE"
        version  = "1.0"
        host     = $hostname
        ip       = $localIP
        universe = $universeTag
        services = $services
        ttl      = 30          # 宣告有效期（秒）
    } | ConvertTo-Json -Compress
}

# 2. UDP 广播发送
function Send-PresenceBroadcast {
    $payload = Build-PresencePayload
    $udp = New-Object System.Net.Sockets.UdpClient
    $data = [System.Text.Encoding]::UTF8.GetBytes($payload)
    $endpoint = New-Object System.Net.IPEndPoint(
        [System.Net.IPAddress]::Parse($BroadcastAddress), $BroadcastPort
    )
    $udp.Send($data, $data.Length, $endpoint) | Out-Null
    $udp.Close()
    Write-Host "   [广播] 主权宣告已发送: $hostname @ $localIP"
}

# 3. 周期性宣告（保持存在感）
$interval = 30  # 秒
while ($true) {
    Send-PresenceBroadcast
    Start-Sleep -Seconds $interval
}
```

### 广播数据载荷设计（JSON 格式重建）

假设 R1 使用的广播数据格式：

```json
{
  "type": "R1_PRESENCE",
  "version": "1.0",
  "host": "LAOZHANG-PC",
  "ip": "192.168.1.100",
  "universe": "WIN",
  "services": {
    "gateway": "3000",
    "bridge": "8080"
  },
  "ttl": 30,
  "signature": "hmac-..."   ← 可选：轻量签名防止伪造广播
}
```

**字段含义**：
- `type`: 协议标识，用于接收端过滤
- `host`/`ip`: 节点身份，接收端可据此建立直连
- `universe`: 宇宙标识（决定接收端如何对齐）
- `services`: 可用端口列表，接收端据此确定连接目标
- `ttl`: 宣告有效期，超时未收到新宣告则标记节点离线
- `signature`: 可选防伪签名（如果 R1 使用了密钥）

## 可达性模式研判：广播发现 → 直连（大概率）

基于现有证据，R1 的可达性模式判断：

### 模式 A：广播发现 → 直连（最可能 ✅）

```
[节点 A] --UDP广播--> [子网内所有节点]
                     ↓ 节点 B 收到广播
                     ↓ 解析 A 的 IP + 端口
[节点 B] --TCP直连--> [节点 A:3000/8080]
```

**证据链**：
1. `Network_Bridge.ps1` 的端口映射是 `Local Port 3000 (Gateway) <-> LAN Port 8080`——典型的直连叙事
2. 没有隧道/代理相关的脚本痕迹（05_LINKS 下只有 bridge 和 broadcast）
3. R1 在内网场景运行，直连是默认假设
4. "主权宣告"——宣告后对方直接来"觐见"，逻辑通顺

### 模式 B：广播发现 → 隧道穿透（不太可能）

```
[节点 A] --UDP广播--> [子网内所有节点]
                     ↓ 节点 B 收到广播
                     ↓ B 通过 A 的 IP 建立某种隧道
```

R1 没有引入 WireGuard/OpenVPN/SSH Tunnel 的痕迹。`Setup-Penetration` 函数的"隧道"描述实际上只是端口映射口径，不是真正的 VPN/隧道协议。

### 模式 C：广播发现 + 代理注册（中等可能）

```
[节点 A] --UDP广播--> [内网 Registry/代理节点]
                     ↓ Registry 记录 A 的存在
                     ↓ 其他节点向 Registry 查询
```

如果 R1 有多节点、多宇宙协同场景，可能需要一个中心注册点。但现有代码中无此痕迹。

## 架构还原：R1 内网可达性全貌

```
                        R1 节点
                    ┌─────────────────┐
                    │  01_KERNEL_MOUNT │── 安全策略门禁
                    │  (security_policy.js)
                    │                 │
                    │  05_LINKS       │
                    │  ├─ Network_Bridge.ps1  → 桥建立（3000 ↔ 8080）
                    │  └─ win_broadcast.ps1   → 广播宣告（UDP:45911）
                    │                 │
                    │  Services       │
                    │  Port 3000 (Gateway) ← 公网/内网入口
                    │  Port 8080 (LAN Service) ← 内网服务
                    └─────────────────┘
                            │
                            ▼ UDP Broadcast (255.255.255.255:45911)
                    ┌──────────────────┐
                    │  内网其他节点     │
                    │  (其他 R1 / 监听器)│← 收到广播后直接连接
                    └──────────────────┘
```

## 对矿场的启示

1. **可引入类似 UDP 广播做 LAN 自动发现**
   - 当前矿场公网桥（zrok + Worker）覆盖了公网入口，但内网节点发现是盲区
   - 可以借鉴 R1 的广播宣告模式，在同局域网内实现"零配置发现"

2. **广播数据格式可复用为心跳/健康检查**
   - R1 的 `ttl` 字段是很好的设计：超时未收到宣告 = 节点可能离线
   - 矿场 Worker 可以加入类似的心跳协议，对内网节点做健康追踪

3. **签名防伪**
   - 如果 R1 的广播确实带了 `signature`，这是值得引入的——UDP 广播天然无认证
   - 矿场内网场景也可以用共享密钥做 HMAC 签名

## 局限性说明

1. `win_broadcast.ps1` 源码未被固化到仓库，以上为基于上下文的**架构重建**而非源码复刻
2. 具体广播端口 `45911` 和 JSON 载荷格式是合理推断，非从源码提取
3. 实际实现的细节（如是否多线程监听、是否同时做广播和监听）已不可考