# Alpine for Alibaba Cloud (ECS)

这是一个面向阿里云 ECS 的 Alpine 自定义镜像项目。
由于官方未直接提供可用的阿里云 Alpine 镜像，这里提供可上传的 `qcow2` 镜像版本。

## 适用场景

- 需要在阿里云 ECS 使用 Alpine Linux
- 需要一个轻量、可通过 cloud-init 初始化的基础镜像
- 需要国内可用的软件源配置（已切换阿里源）

---

## 使用说明（阿里云）

1. 从 Releases 下载 `alpine-custom.qcow2`
2. 在阿里云导入自定义镜像
3. 创建 ECS 时可绑定 SSH 密钥对（镜像内已内置公钥，非必需）
4. 系统盘最小选择 **1G 即可**

---

## 构建镜像（GitHub Actions）

镜像由 `.github/workflows/build-alpine-image.yml` 构建，触发方式二选一：

- 推送 `v*` 形式的 tag：
  ```bash
  git tag v2.0.2 && git push origin v2.0.2
  ```
- 在仓库 Actions 页面选择 `Build Alpine Cloud Image` → **Run workflow** 手动触发，可填写要发布的 release tag，留空则用 `manual-<run number>`

构建完成后：

- 镜像以 `alpine-custom.qcow2` 发布到 GitHub Release（tag 触发用该 tag，手动触发用指定 tag 或 `manual-<run number>`）
- 同时作为 workflow artifact 保留 14 天，可在对应 run 页面直接下载

发布 Release 使用 GitHub 内置的 `GITHUB_TOKEN`，不需要额外配置 secret。

> 仓库内仍保留 `.circleci/config.yml`。那套流水线是 CircleCI 专用的，需要在 circleci.com 单独接入本仓库后才会运行；未接入则不会触发。

---

## 登录与权限（重要）

- 镜像内已预置 SSH 公钥，`root` 与 `alpine` 两个用户均可直接用对应私钥登录
- 直接用 root 登录：
  ```bash
  ssh -i <你的私钥> root@<ECS公网IP>
  ```
- 也可用普通用户登录后提权：
  ```bash
  ssh -i <你的私钥> alpine@<ECS公网IP>
  sudo -i
  ```
- 已启用密码登录，但镜像内**不预设任何密码**，且禁止空密码登录（`PermitEmptyPasswords no`）；因此默认仍只能用密钥登录，需要密码登录时请登录后自行 `passwd` 设置

> 创建实例时额外绑定密钥对同样受支持，cloud-init 会把绑定密钥追加进去。

---

## 镜像已做的定制

- 启用 cloud-init（支持云端初始化）
- 配置 growpart / resize_rootfs（首启可自动扩容到系统盘大小）
- 预装常用基础组件：`cloud-init`、`chrony`、`sudo` 等
- SSH 配置（支持密钥登录；允许密码登录但不预设密码，禁止空密码）
- 内置 SSH 公钥（`root` 与 `alpine` 用户），无需绑定密钥对即可登录
- 时区设置
- 优化部分内核/网络参数
- 默认软件源已切换为阿里云镜像源（便于国内使用）

---

## 镜像未包含内容

- 不包含阿里云官方 Agent（如云助手等）
- 不包含额外业务软件栈（Docker/K8s/监控等）

如有需要，请在实例初始化后自行安装。

---

## 免责声明

本镜像为社区用途的自定义构建版本，请先在测试环境验证后再用于生产环境。
