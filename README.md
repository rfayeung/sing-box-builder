# sing-box IPK for GL-AXT1800

GitHub Actions 自动编译 sing-box ipk 插件，适用于 **GL.iNet AXT1800** (固件 v4.8.2, 架构 aarch64_cortex-a53, 内核 5.4.164)。

## 快速开始

### 方案 A：使用 GitHub Actions 自动编译（推荐）

1. **Fork/创建仓库**：将此仓库推送到 GitHub
2. **触发编译**：
   - 进入 `Actions` → `Build sing-box IPK for GL-AXT1800` → `Run workflow`
   - 可选填版本号（如 `v1.14.0-alpha.26`），留空自动使用最新 release
3. **下载 IPK**：编译完成后在 Artifacts 中下载 `.ipk` 文件
4. **安装到路由器**：

```bash
scp sing-box_*.ipk root@192.168.8.1:/tmp/
ssh root@192.168.8.1
opkg update && opkg install /tmp/sing-box_*.ipk --force-depends
```

> `--force-depends` 跳过 kmod 内核版本检查（你的固件内核为 5.4.164，与主线 OpenWrt 不同）

### 方案 B：直接下载官方预编译包（最快）

```bash
cd /tmp
wget https://github.com/SagerNet/sing-box/releases/download/v1.14.0-alpha.26/sing-box_1.14.0-alpha.26_openwrt_aarch64_cortex-a53.ipk
opkg update && opkg install sing-box_*.ipk --force-depends
```

最新版本见 [releases](https://github.com/SagerNet/sing-box/releases)

### 方案 C：本地 Go 交叉编译

```bash
git clone https://github.com/SagerNet/sing-box.git
cd sing-box
GOOS=linux GOARCH=arm64 CGO_ENABLED=0 go build \
  -ldflags="-s -w -X github.com/sagernet/sing-box/constant.Version=$(git describe --tags)" \
  -tags "with_quic,with_wireguard,with_utls,with_clash_api,with_gvisor,with_dhcp,with_tailscale" \
  -o sing-box ./cmd/sing-box
```

然后将二进制手动打包为 ipk（参考 workflow 文件中的步骤）。

## 编译方案说明

本仓库使用 **Go 交叉编译 + IPK 打包** 方案，而非完整的 OpenWrt SDK 编译，原因：

| 方案 | 优点 | 缺点 |
|------|------|------|
| **Go 交叉编译** (本方案) | 5分钟完成，不依赖 SDK | 缺少 kmod 版本校验 |
| OpenWrt SDK 全量编译 | 完美兼容性 | 2+小时，依赖已关闭的 GL SDK |
| 官方预编译包 | 即下即用 | 版本由上游控制 |

> GL.iNet 的 `gl-infra-builder` SDK 仓库已于 2024 年底转为私有仓库。本方案使用 `CGO_ENABLED=0` 纯 Go 编译，生成的静态二进制文件与 OpenWrt musl 环境完全兼容。

## 安装后配置

```bash
# 编辑配置
vi /etc/sing-box/config.json

# 启动服务
/etc/init.d/sing-box enable
/etc/init.d/sing-box start

# 查看状态
logread -e sing-box
```
