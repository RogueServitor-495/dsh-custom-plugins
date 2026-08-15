# AGENTS.md — dsh-custom-plugins 仓库规范

本仓库是 **dsh（DeepSeek Harness）自定义插件**的父仓库（meta-repo）。

## 核心约定：一个插件 = 一个子模块

> **每个插件都是独立的 git 仓库（submodule），挂载在本父仓库下。父仓库本身只包含规范文档（AGENTS.md / README）和子模块引用，不直接存放插件代码。**

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
