# AGENTS.md — dsh-custom-plugins 仓库规范

本仓库是 **dsh（DeepSeek Harness）自定义插件**的父仓库（meta-repo）。

## 核心约定：一个插件 = 一个子模块

> **每个插件都是独立的 git 仓库（submodule），挂载在本父仓库下。父仓库本身只包含规范文档（AGENTS.md / README）和子模块引用，不直接存放插件代码。**

## Git 项目管理规范（强制）

> **父仓库 `dsh-custom-plugins` 是所有插件的唯一收纳与管理入口：所有插件必须以子模块（submodule）形式挂载在父仓库下。** 父仓库的 index 中必须有对应 gitlink、`.gitmodules` 中必须有对应条目；仅存在插件目录而未注册子模块，即视为「未收编」状态，**禁止直接提交/推送，必须先按下方流程收编**。

### 一、插件生命周期 git 规则

1. **一个插件 = 一个独立 git 仓库 + 父仓库里的一个子模块**。插件代码只存在于插件仓库；父仓库只记录子模块指针与规范文档，不直接存放插件代码。
2. **新插件上线必经流程**（缺一不可）：
   1. 在 GitHub 建仓 `<owner>/dsh-plugin-<name>.git`（与父仓库同 owner，可先建空仓）；
   2. 插件代码完成开发并本地提交（`git init -b main` → `git add -A` → `git commit`）；
   3. 推送插件仓库到 GitHub（`git push -u origin main`）；
   4. 在父仓库注册子模块并提交（见「二、注册子模块」）；
   5. 同步更新 `README.md` 插件清单；
   6. 推送父仓库（`git push origin main`）。
3. **已开发但未注册子模块的插件 → 收编流程**：开发期允许插件先以普通目录存在，但任何提交/推送前必须完成收编：
   - 本地 `git init -b main` 初始化插件仓库，核对 `.gitignore`（必须含 `dsh-home/`、`node_modules/`、`*.log`、`.DS_Store`），确认无密钥后提交；
   - GitHub 建仓 → 推送插件仓库（先于父仓库提交，保证协作者 `git submodule update --init` 能拉到）；
   - 父仓库注册子模块 → 提交 → 推送父仓库。
4. **子模块内部有未提交改动（`modified content`）时**：改动一律先在子模块内「提交 + 推送」，再在父仓库 bump 子模块指针并提交推送；**禁止**长期保留脏子模块，也不得把子模块改动绕道提交进父仓库。
5. **父仓库工作区必须保持干净**：
   - `.DS_Store`、`*.log` 等由父仓库 `.gitignore` 屏蔽，禁止提交；
   - 提交前必查 `git status`：只允许「文档变更 + 子模块指针变更（新增 gitlink / `modified: <plugin> (new commits)`）」，其余一律清理。
6. **提交信息**：遵循 `type: 描述` 前缀（`feat:` / `fix:` / `chore:` / `docs:`）；新增子模块用 `chore: add <plugin-name> submodule`，指针更新用 `chore: bump <plugin-name> 子模块`。

### 二、注册子模块（已有本地插件仓库时）

插件目录已在本地（已 git init 并提交）时，无需重新 clone，直接注册：

```sh
cd dsh-custom-plugins
# 1) 写入 .gitmodules（相对 URL，GitHub 自动解析为同 owner 兄弟仓库）
git config -f .gitmodules submodule.<plugin-name>.path <plugin-name>
git config -f .gitmodules submodule.<plugin-name>.url ../<plugin-name>.git
# 2) 把插件当前提交注册为 gitlink
git update-index --add --cacheinfo 160000,$(git -C <plugin-name> rev-parse HEAD) <plugin-name>
# 3) 本地初始化子模块记录，提交并推送
git submodule init
git add .gitmodules <plugin-name>
git commit -m "chore: add <plugin-name> submodule"
git push origin main
```

> ⚠️ 插件仓库必须**先**建仓并推送，否则其他协作者 `git submodule update --init` 会拉取失败。

### 三、收编检查清单（提交父仓库前逐项核对）

- [ ] 插件目录已注册：`.gitmodules` 有条目，父仓库 index 有 gitlink（`git submodule status` 无 `-` 前缀）；
- [ ] 插件仓库已推送 GitHub，且与 `.gitmodules` 中 URL（同 owner）一致；
- [ ] 插件 `.gitignore` 含 `dsh-home/`、`node_modules/`、`*.log`、`.DS_Store`；`git grep -n "sk-" <plugin> HEAD` 无密钥；
- [ ] `README.md` 插件清单已补新插件；
- [ ] `git status` 仅剩预期变更（文档 + 子模块指针）。

## 插件命名规范

- **统一前缀**：所有插件目录名一律使用 `dsh-plugin-<name>` 格式（如 `dsh-plugin-speaker`、`dsh-plugin-voice-input`、`dsh-plugin-deepseek-usage`），不使用 `dsh-<name>` / `tts` 等其他前缀。
- **名字全局一致**：目录名、`package.json` 的 `name`、插件注册名（`export const name`）、浏览器 bundle 模块 `id`、settings namespace 必须与 `dsh-plugin-<name>` 完全一致（`cordis.patch.yml` 的行 `id` / `name` 也使用同一名字）。
- **仓库同名**：插件 GitHub 仓库与目录名一致：`<owner>/dsh-plugin-<name>.git`；`.gitmodules` 的 `path` / `url` 均使用该名字。

### 目录结构

```
dsh-custom-plugins/                 # 父仓库（本仓库）
├── AGENTS.md                       # 本文件：仓库规范
├── README.md                       # 仓库总览（插件清单、快速上手）
└── <plugin-name>/                  # 每个插件一个子模块
    ├── package.json                # 插件包（含 dsh.client 声明）
    ├── lib/                        # 服务端 half（node）+ 浏览器 half（client bundle）
    ├── README.md                   # 插件自身说明
    ├── dev/                        # 独立开发实例脚本（start.sh）
    ├── dsh-home/                   # 开发用独立 DSH home（含凭据/会话，**必须 gitignore**）
    └── ...
```

### 子模块 URL 约定

- 所有插件仓库与父仓库位于**同一 GitHub 账号/组织**下。
- `.gitmodules` 使用**相对 URL**：`url = ../<plugin-name>.git`（GitHub 会自动解析为同 owner 下的兄弟仓库），例如：

```ini
[submodule "dsh-plugin-deepseek-usage"]
    path = dsh-plugin-deepseek-usage
    url = ../dsh-plugin-deepseek-usage.git
```

## 新增一个插件

1. 在 GitHub 为插件创建独立仓库：`<owner>/<plugin-name>.git`（与父仓库同 owner）。
2. 把插件代码作为该仓库的默认分支（`main`）提交。
3. 在父仓库注册为子模块：

```sh
git submodule add ../<plugin-name>.git <plugin-name>
git commit -m "chore: add <plugin-name> submodule"
git push
```

4. 在 `README.md` 的插件清单补一行（插件名 + 一句话说明）。
5. 拉取时用 `git submodule update --init --recursive`（或 `--remote` 更新到各插件最新提交）。

## 插件开发规范

### 双 half 结构（服务端 + 浏览器端）

- 服务端：`lib/index.js` 导出 `{ name, inject, apply }` 的 cordis 插件，注册 HTTP 路由 / 服务 / 投影等。
- 浏览器端：包 `package.json` 声明 `dsh.client: { platform: "web" }`，`exports["./client"]` 指向浏览器 bundle（`window.__ModuleLoader__.load({ id, factory })` 格式）。
- 浏览器端默认从**同源 API 路由**取数据；API key 等敏感信息只走服务端（凭据 seam `ctx.credentials`），**绝不**下发到浏览器。

### 安装到 dsh profile

```sh
# 1) 装进 profile 的 node_modules（dsh 会把相对路径锚定到当前目录）
dsh plugin --profile web add file:/absolute/path/to/<plugin>

# 2) 在 ~/.dsh/profiles/<name>/cordis.patch.yml 追加：
- insert:
    - id: <row-id>
      name: '<package-name>'

# 3) 服务端行会通过 patch 监听热加载；浏览器端整页刷新后生效（新增客户端包需重启才被扫描）
```

### 独立开发实例（推荐）

每个插件可自带 `dev/start.sh`，用插件目录内 gitignore 的 `dsh-home/` 启动一个不改动 `~/.dsh` 的独立 web 实例：

```sh
bash dev/start.sh        # 默认 http://127.0.0.1:3081
```

`dsh-home/` 由 `dev/setup-home.sh`（或首次启动脚本）从主环境复制：`.credentials.yaml`、`settings.yaml`、`sessions/` 快照等。

## 安全红线（必须遵守）

- ❌ **禁止提交任何密钥**：`.credentials.yaml`、`DEEPSEEK_API_KEY` 等凭据内容。
- ❌ **禁止提交运行时数据**：`dsh-home/`（含会话日志、storages）、`node_modules/`、日志文件。
- ✅ 每个插件仓库必须含 `.gitignore`：`dsh-home/`、`node_modules/`、`*.log`、`.DS_Store`。
- ✅ 提交前检查：`git grep -n "sk-" HEAD`、`git status` 确认无敏感文件。
- ✅ 代码中 API key 一律通过 dsh 凭据 seam 运行时解析，不写死、不落库。

## 提交规范

- 提交信息用中文或英文均可，遵循 `type: 描述` 前缀（`feat:` / `fix:` / `chore:` / `docs:`）。
- 每个子模块独立提交、独立版本；父仓库提交只记录子模块指针变化与文档变更。

## 浏览器端验证清单（改 UI 后必做）

1. `node --check lib/client.js` 语法检查。
2. 启动独立实例，Playwright/浏览器打开页面，确认渲染元素存在、无 console/page error。
3. 确认 `GET /api/...` 返回真实数据（余额等）且错误态可展示。
4. 确认刷新按钮 / 自动刷新 / 页面可见时刷新均触发请求。
