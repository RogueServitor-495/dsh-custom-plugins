# dsh-custom-plugins

dsh（DeepSeek Harness）自定义插件父仓库。**仓库规范见 [AGENTS.md](./AGENTS.md)：一个插件一个子模块**。

## 插件清单

| 插件 | 说明 | 仓库 |
|---|---|---|
| [dsh-plugin-speaker](./dsh-plugin-speaker) | 文字转语音播报（speak / tts_voices；macOS say+afplay / Windows SAPI） | `RogueServitor-495/dsh-speaker` |
| [dsh-plugin-deepseek-usage](./dsh-plugin-deepseek-usage) | 在 Web 页面顶端展示 DeepSeek API 当日 token 使用量 ｜ 计费 ｜ 余额 | `RogueServitor-495/dsh-plugin-deepseek-usage` |
| [dsh-plugin-voice-input](./dsh-plugin-voice-input) | 语音转文字输入按钮（Web Speech API 🎤，客户端插件） | `RogueServitor-495/dsh-plugin-voice-input` |

## 快速上手

```sh
git clone git@github.com:RogueServitor-495/dsh-custom-plugins.git
cd dsh-custom-plugins
git submodule update --init --recursive   # 拉取各插件子模块
```

安装插件到 dsh profile：见各插件 README，以及 [AGENTS.md](./AGENTS.md) 的「安装到 dsh profile」。
