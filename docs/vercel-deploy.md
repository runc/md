# Vercel 部署（`feat_vercel`）与发布流程

本文档记录本仓库为支持 Vercel 部署做的改动，以及后续推荐的发布/同步流程。

## 目标与分支约定

- `main`：不做任何开发写操作，仅用于与上游（`upstream/main`）保持同步。
- `feat_vercel`：本次功能分支，用于添加/维护 Vercel 部署支持。

## 本次改动点

### 1) 新增 `vercel.json`

- 使用 `@vercel/static-build` 构建静态站点
- 构建输出目录：`apps/web/dist`
- 安装/构建命令：
  - `installCommand`：通过 `npx -y pnpm@10.24.0 ...` 固定 pnpm 版本（避免 Vercel 默认 pnpm 版本与项目不一致导致的问题）
  - `buildCommand`：`pnpm web build:only`

说明：由于 `vercel.json` 里使用了 `builds`，Vercel 控制台里配置的 Build/Install/Output 等设置不会生效（这是 Vercel 的正常行为）。

### 2) 调整 `apps/web/vite.config.ts`

Vite 的 `base` 在本项目默认是 `/md/`（用于部署到子路径）。

在 Vercel 环境（`VERCEL=1`）下，将 `base` 设为 `/`，确保 `index.html` 中静态资源引用路径为 `/static/...`，从而在根路径部署可用。

### 3) 更新 `README.md`

补充 Vercel 部署说明入口（根目录 `/` 部署）。

## 发布（部署）方式

### 方式 A：Vercel 控制台（推荐）

1. Vercel 控制台 `New Project` → 选择该 GitHub 仓库（fork）。
2. 选择需要发布的分支：
   - 想让 `feat_vercel` 直接作为生产环境：将 Production Branch 设为 `feat_vercel`。
   - 想保留“生产=main、预览=feat\_\*”的惯例：让 `feat_vercel` 作为 Preview Deployments 分支（通过 PR/Push 触发预览）。
3. 部署完成后，使用 Vercel 分配的域名访问。

### 方式 B：Vercel CLI（适合本地手动发布）

前提：本机已登录 `vercel`（`vercel whoami` 能看到账号）。

如果你需要走代理，可先导出环境变量：

```bash
export https_proxy=http://127.0.0.1:7890
export http_proxy=http://127.0.0.1:7890
export all_proxy=socks5://127.0.0.1:7890
```

部署到 Production：

```bash
vercel deploy --prod
```

注意：

- `vercel link/deploy` 可能会在本地生成 `.vercel/`、并修改 `.gitignore` / 生成 `.env.local`（取决于你的项目设置）。
- 如果你不希望这些变动进入 Git 工作区，可以在临时目录部署（例如通过 `git archive` 导出后在临时目录执行 `vercel` 命令），或在部署后丢弃本地变更。

## 与上游同步（`main` 永远保持干净）

1. 添加上游（仅需一次）：

```bash
git remote add upstream https://github.com/doocs/md.git
git fetch upstream
```

2. 同步本地与远端 `main`（只允许快进，避免产生 merge commit）：

```bash
git checkout main
git fetch upstream
git merge --ff-only upstream/main
git push origin main
```

3. 将上游更新带入功能分支（推荐 rebase 保持历史线性）：

```bash
git checkout feat_vercel
git rebase main
git push --force-with-lease
```
