# dsh-custom-plugins

dsh（DeepSeek Harness）自定义插件父仓库。**仓库规范见 [AGENTS.md](./AGENTS.md)：一个插件一个子模块**。

## 插件清单

| 插件 | 说明 | 仓库 |
|---|---|---|
| [dsh-plugin-speaker](./dsh-plugin-speaker) | 文字转语音播报（speak / tts_voices；macOS say+afplay / Windows SAPI） | `RogueServitor-495/dsh-plugin-speaker` |
| [dsh-plugin-deepseek-usage](./dsh-plugin-deepseek-usage) | 在 Web 页面顶端展示 DeepSeek API 当日 token 使用量 ｜ 计费 ｜ 余额 | `RogueServitor-495/dsh-plugin-deepseek-usage` |
| [dsh-plugin-manager](./dsh-plugin-manager) | 插件管理：列表/启停/移除已有插件，通过 git URL / npm / tarball / 本地路径导入新插件（Web 面板 + 同源 API） | `RogueServitor-495/dsh-plugin-manager` |
| [dsh-plugin-voice-input](./dsh-plugin-voice-input) | 语音转文字输入按钮（Web Speech API 🎤，客户端插件） | `RogueServitor-495/dsh-plugin-voice-input` |
| [dsh-plugin-chat-display](./dsh-plugin-chat-display) | 对话展示样式：设置 → 通用 可视化预设卡片（面板 / 原生 / 卡片 / 紧凑），带演示缩略图 | `RogueServitor-495/dsh-plugin-chat-display` |
| [dsh-plugin-context-view](./dsh-plugin-context-view) | 上下文+工作区管理面板：页签切换——上下文（指令文件/Skills/工具MCP/规范文档存在与内容双维）+ 工作区（目录树/打开方式/简易编辑器） | `RogueServitor-495/dsh-plugin-context-view` |
| [dsh-plugin-context-overview](./dsh-plugin-context-overview) | 对话状态数据 + macOS 系统级原生悬浮框（独立于浏览器，可插件托管自动拉起） | `RogueServitor-495/dsh-plugin-context-overview` |
| [dsh-plugin-knowledge-base](./dsh-plugin-knowledge-base) | 跨对话知识库：飞书 Wiki 文档检索与收藏（Agent 工具 + 浏览器管理面板，凭据/收藏 UI 配置） | `RogueServitor-495/dsh-plugin-knowledge-base` |
| [dsh-plugin-todo-list](./dsh-plugin-todo-list) | 轻量待办清单：悬浮按钮 + 抽屉面板，添加/完成/内联编辑/删除/截止日期/筛选/进度，JSON 文件持久化 | `RogueServitor-495/dsh-plugin-todo-list` |

## 快速上手

```sh
git clone git@github.com:RogueServitor-495/dsh-custom-plugins.git
cd dsh-custom-plugins
git submodule update --init --recursive   # 拉取各插件子模块
```

安装插件到 dsh profile：见各插件 README，以及 [AGENTS.md](./AGENTS.md) 的「安装到 dsh profile」。
