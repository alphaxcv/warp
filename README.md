# Setup WARP Action

在 GitHub Actions 上一键安装并连接 **Cloudflare WARP** 的可复用 composite action。

采用**普通 warp 全隧道模式**（非 `warp+doh`），因此 `warp-cli registration new` 有效——可以在运行中**重新注册以轮换出口 IP**。

## 用法

> 前提：调用前必须先 `actions/checkout`（本地路径调用 `uses: ./` 时需要仓库已检出）。

### 在本仓库内调用

```yaml
steps:
  - uses: actions/checkout@v4

  - name: Setup Cloudflare WARP
    uses: ./
    with:
      stack: dual
```

### 在其它仓库调用

把本仓库当作 action 仓库引用：

```yaml
steps:
  - name: Setup Cloudflare WARP
    uses: alphaxcv/warp@1.0
    with:
      stack: dual
```

## 输入参数

| 参数 | 必填 | 默认 | 说明 |
| --- | --- | --- | --- |
| `stack` | 否 | `dual` | IP 栈：`ipv4` / `ipv6` / `dual`（双栈） |

## 行为说明

该 action 会依次：

1. 安装 `cloudflare-warp`；
2. `registration new` 注册新账号；
3. 按 `stack` 限定 IP 栈（`dual` 不限制）；
4. `connect` 连接，最多重试 8 次直到状态变为 `Connected`；
5. 打印当前 WARP 出口 IP 与 `cdn-cgi/trace` 信息。

连接成功后，WARP 为**系统级全隧道**，该 runner 上的所有流量（包括浏览器与 `requests`）自动经 WARP 出口，无需在程序里单独设置代理。

## 运行中轮换 IP

由于使用普通 warp 模式，可在脚本里通过重新注册来更换出口 IP：

```bash
sudo warp-cli --accept-tos disconnect
sudo warp-cli --accept-tos registration delete
sudo warp-cli --accept-tos registration new
sudo warp-cli --accept-tos connect
```

## 注意

- 仅适用于 `ubuntu-*` runner。
- WARP 免费出口 IP 段（如 `104.28.x`）可能被部分站点/风控（例如 Google reCAPTCHA）降权，IP 可轮换不代表一定能绕过这类风控。
- 若需更干净的出口，可在 `registration new` 后用 `registration license <KEY>` 绑定 WARP+ / Zero Trust 授权。
