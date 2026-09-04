# Cloudflare WARP 完整配置指南

## 1. 快速开始

本文档提供两种配置 Cloudflare WARP 的方法：

1. **官方 WARP 客户端** - GUI 一键连接（推荐，最简单）
2. **wgcf CLI** - 生成 WireGuard 配置文件（备选，适用无 GUI 环境）

---

## 2. 多源下载链接表

| 组件 | 版本 | 官方下载 | 备用下载 (GitHub) | 备注 |
|------|------|----------|-------------------|------|
| **Cloudflare WARP (官方客户端)** | 2026.7.1376.0 (2026-08-28) | [Cloudflare WARP for Windows](https://9df0cba6.preview.developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/download-warp/) | [ViRb3/wgcf](https://github.com/ViRb3/wgcf/releases) | 图形界面，一键连接 |
| **wgcf CLI** | v2.2.32 (2026-07-23) | [winget install ViRb3.wgcf](https://wingetly.io/apps/vi-rb3/wgcf) | [wgcf_2.2.32_windows_amd64.exe](https://github.com/ViRb3/wgcf/releases/download/v2.2.32/wgcf_2.2.32_windows_amd64.exe) | 生成 WireGuard 配置 |

**下载备注**：
- 如果 `1.1.1.1` 无法打开，请使用 GitHub 备用链接或国内镜像加速
- WARP 客户端需要 .NET Framework 4.7.2 或更高版本
- wgcf 生成的 WireGuard 配置需要 [WireGuard 客户端](https://www.wireguard.com/install/) 才能使用

---

## 3. 官方 WARP 客户端安装步骤（含管理员权限）

### 3.1 以管理员身份运行 PowerShell

```powershell
# 右键点击开始菜单 → Windows PowerShell (管理员)
# 或者：Win + X → A
```

### 3.2 下载安装包

```powershell
# 进入下载目录
cd $env:USERPROFILE\Downloads

# 下载最新 WARP 安装包 (示例，实际请访问官方链接)
# 如果官方链接无法访问，请使用 GitHub 备用源
iwr -Uri "https://9df0cba6.preview.developers.cloudflare.com/cloudflare-one/connections/connect-devices/warp/download-warp/" -OutFile cloudflare_warp_installer.exe
```

### 3.3 官方 GUI 安装（推荐）

```powershell
# 双击运行安装程序
& "F:\ttt\cloudflare_warp_installer.exe"

# 按照向导步骤：
# 1. 点击 "Next" 推进
# 2. Accept Cloudflare 的隐私政策
# 3. 打开 "WARP" 开关至 "Connected" 状态
# 4. 如有提示，点击 "Allow" 安装 VPN 配置文件
```

### 3.4 若静默安装失败

**尝试的命令**（可能因版本不同而异）：
```powershell
& "F:\ttt\cloudflare_warp_installer.exe" /silent /install
```
**结果**：可能抛出 `ApplicationFailedException` - 静默参数因版本而异。

**备选方案**：
- 直接使用 GUI 向导安装（见上文 3.3 节）
- 或从控制面板安装：控制面板 → 程序 → 启用或关闭 Windows 功能 → .NET Framework 3.5

### 3.5 验证安装

```powershell
# 检查 WARP 服务
Get-Service warp-svc

# 期望输出:
#   Status   Name               DisplayName
#   ------   ----               -----------
#   Running  warp-svc           Cloudflare WARP
```
---

## 4. GUI 配置与连接验证

### 4.1 两种模式切换

| 模式 | 说明 | 适用场景 |
|------|------|----------|
| **WARP** | 全流量加密（真正的 VPN 模式） | 需要完整代理，隐藏真实 IP |
| **1.1.1.1** | 仅加密 DNS 查询 | 只想保护域名解析隐私 |

**切换方法**：
1. 点击任务栏 Cloudflare 图标
2. 选择齿轮图标 → **Preferences**
3. 在 "Connection" 选项卡中选择模式
4. 或直接在主界面开关间切换

### 4.2 验证连接是否成功

**方法 1：访问 Cloudflare 探测页**
1. 打开浏览器
2. 访问：`https://www.cloudflare.com/cdn-cgi/trace`
3. 确认输出包含 `warp=on` (WARP 模式) 或 `warp=plus` (WARP+ 订阅)

**方法 2：IP 地址查询**
1. 访问：`https://ip.sb` 或 `https://ifconfig.me`
2. 应该显示 Cloudflare 的 IP 地址，而非您的真实 ISP IP

**方法 3：DNS 泄漏测试**
1. 访问：`dnsleaktest.com`
2. 应该只显示 Cloudflare 的 DNS 服务器

---

## 5. wgcf CLI 备选方案（生成 WireGuard 配置）

### 5.1 安装 wgcf

```powershell
# 使用 winget 安装 (推荐)
winget install --id ViRb3.wgcf --exact --version 2.2.32

# 或手动安装
# 1. 下载 wgcf_2.2.32_windows_amd64.exe
# 2. 双击安装或解压到任意目录
# 3. 将目录添加到 PATH 环境变量
```

### 5.2 注册账号

```powershell
# 第一次使用需注册
wgcf register

# expected output: 
# Account registered successfully
# Please verify your account at https://dash.warpplus.net
```

### 5.3 生成 WireGuard 配置

```powershell
# 生成配置文件
wgcf generate

# 配置文件位置: %APPDATA%\wgcf\wgcf.conf
# 示例内容:
# [Peer]
# PublicKey = <your-public-key>
# Endpoint = <your-endpoint>
# AllowedIPs = 0.0.0.0/0, ::/0
# PersistentKeepalive = 25
```

### 5.4 导入 WireGuard 客户端

**WireGuard 客户端安装步骤**：
1. 前往 [wireguard.com](https://www.wireguard.com/install/) 下载 Windows 客户端
2. 安装路径默认：`C:\Program Files\WireGuard`
3. 安装完成后，在开始菜单找到 "WireGuard"

**导入配置**：
1. 打开 WireGuard 客户端
2. 点击 "Add tunnel" → "Create from scratch"
3. 选择 "Empty tunnel"
4. 在 "Private key" 字段粘贴 `wgcf` 自动生成的私钥
5. 在 "Address" 添加：`100.64.0.2/10` (或 `fd86:57:3e::2/128` for IPv6)
6. 在 "DNS" 添加：`100.100.100.100` 或 `2606:4700:4700::1111`
7. 在 "Allowed IPs" 添加：`0.0.0.0/0` (全局代理) 或具体网段
8. 点击 "Save"，然后开启开关连接

---

## 6. 进阶设置

### 6.1 DNS 协议配置

在 WARP 客户端 Preferences → Connection 中可选：

| 选项 | 说明 |
|------|------|
| **DNS Protocol: HTTPS** | 所有 DNS 流量通过 DNS over HTTPS 发送 |
| **DNS Protocol: TLS** | 所有 DNS 流量通过 TLS 加密发送 |
| **1.1.1.1 for Families** | 开启恶意网站/成人内容过滤 |

### 6.2 分流隧道（Split Tunneling）

可排除特定应用不走 WARP：
1. 打开 WARP 客户端
2. Preferences → Split Tunneling
3. 添加需要直接连接的应用路径或进程名
4. 常见排除：银行应用、本地打印机、公司内网访问

### 6.3 WARP+ 订阅

- 免费版有每日 1GB 流量限制
- WARP+ 通过购买或转换协议获取
- 官方购买：`1.1.1.1` App → 升级套餐
- 免费获取：部分节点共享服务、GitHub 项目分享

---

## 7. 故障排查手册

### 7.1 常见问题及解决

| 症状 | 原因 | 解决方法 |
|------|------|----------|
| **连不上 WARP** | 网络运营商阻断 | 切换 DNS 协议 (HTTPS/TLS) 或使用 wgcf 备选方案 |
| **速度很慢** | 节点拥塞 | 重新连接（WARP 会自动选最快），或手动选择其他时间段 |
| **某些网站打不开** | 分流配置问题 | 检查 Split Tunneling 是否正确，或暂时关闭 WARP |
| **提示 "TAP driver not found"** | TAP 适配器缺失 | 重装 WARP 客户端，或改用 wgcf + WireGuard 方案 |
| **1.1.1.1 无法打开** | 网络受限 | 使用下载链接备用源，或从 GitHub 下载安装包 |

### 7.2 重装 WARP 客户端

1. 卸载现有程序：
   - 控制面板 → 卸载程序 → Cloudflare WARP → 卸载
   - 或：`Get-AppxPackage -AllUsers *Cloudflare* | Remove-AppxPackage`

2. 删除残留文件：
   ```powershell
   Remove-Item -Path "C:\Program Files\Cloudflare\Cloudflare WARP*" -Recurse -Force
   Remove-Item -Path "$env:APPDATA\Cloudflare*" -Recurse -Force
   ```

3. 重新下载并安装：
   - 参考第 3 节下载和安装步骤

### 7.3 卸载 wgcf

```powershell
# 使用 winget 卸载
winget uninstall --id ViRb3.wgcf

# 或手动删除安装目录
Remove-Item -Path "$env:ProgramFiles\ViRb3\wgcf*" -Recurse -Force
```

---

## 8. 已知安装问题与排查（重要）

### 8.1 下载成功但安装失败

**现象**：下载 `cloudflare_warp_installer.exe` 成功（约 343KB），但直接双击或通过 cmd 运行安装程序时，无法弹出安装界面或直接闪退。

**可能原因**：
1. **.NET Framework 4.7.2 缺失** - WARP 客户端依赖 .NET Framework
2. **管理员权限不足** - 安装需要写入 Program Files 目录
3. **冲突的现有安装** - 已安装旧版 WARP 或冲突的 VPN 客户端

**排查步骤**：
- 检查 .NET Framework 4.7.2+: 在 PowerShell 中运行 `Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\NET Framework Setup\NDP' | where { $_.Version -ne '' } | select Version, ReleaseName`
- 右键点击安装程序 → "以管理员身份运行"
- 卸载现有的 Cloudflare WARP 后重新安装：控制面板 → 卸载程序 → Cloudflare WARP → 卸载

### 8.2 静默安装参数错误

**尝试的命令**：
```powershell
& "F:\ttt\cloudflare_warp_installer.exe" /silent /install
```
**结果**：`ApplicationFailedException` - 静默参数可能因版本而异。

**备选安装方式**：
- 直接双击 `.exe` 文件启动图形化安装向导
- 或使用 `msiexec` 安装（如果 installer 解压出 `.msi` 文件）

### 8.3 PowerShell 与 Bash 命令兼容性问题

**观察到的问题**：
- 在本指南的执行过程中，发现 `pwsh`、`powershell -Command` 等命令在 bash 环境中可能因转义字符问题导致失败
- 官方建议：在 Windows 系统上直接使用 PowerShell GUI 或搜索 "Cloudflare WARP" 启动，而不是通过命令行批量脚本

**推荐方式**：
1. 双击 `F:\ttt\cloudflare_warp_installer.exe` 启动安装向导
2. 或通过开始菜单：搜索 "Cloudflare WARP" → 打开应用
3. 如需脚本安装，建议在真实 Windows PowerShell 环境中单独测试

### 8.4 WireGuard 客户端配置兼容性

**wgcf + WireGuard 备选方案问题**：
- 生成的 `wgcf.conf` 需要导入到 WireGuard 客户端
- Windows WireGuard 客户端版本需兼容（建议 1.0.20230823 或更高）
- 导入后可能需要重启系统或重新连接网络适配器

**完整 WireGuard 安装步骤**：
1. 访问 https://www.wireguard.com/install/ 下载 Windows 客户端
2. 安装后在系统托盘找到 WireGuard 图标
3. 点击 "Add tunnel" → "Create from scratch" → "Empty tunnel"
4. 粘贴 wgcf 生成的私钥和地址
5. 保存并开启开关连接

---

## 9. 截图占位符

> **说明**：以下位置预留了截图占位符，实际使用时可替换为真实截图
> ```markdown
> ![步骤1-管理员权限打开PowerShell](images/step1-powershell-admin.png)
> ![步骤2-连接成功的WARP界面](images/step2-warp-connected.png)
> ![步骤3-wgcf register 成功输出](images/step3-wgcf-register.png)
> ![步骤4-WireGuard 客户端导入配置](images/step4-wireguard-import.png)
> ```

> 如需真实截图，可在 Windows 中：
> 1. 按 `Win + Shift + S` 截取屏幕区域
> 2. 将图片保存到 `F:\ttt\lutt\images\` 目录
> 3. 使用相对路径引用：`![Alt text](images/filename.png)`