# 企业微信 + DeepSeek 机器人 🤖

> 基于 chatgpt-on-wechat（现 CowAgent）框架，零封号风险，支持强制人设，零基础可部署。

## ✨ 特性

- ✅ **官方 API，合法合规** — 使用企业微信官方消息回调 API，永远不会封号
- ✅ **强制人设，永久生效** — 修复了原框架 `system_prompt` 不生效的 bug，直接硬编码人设
- ✅ **零基础可部署** — 配套 487 行图文教程，从 Python 安装到机器人上线，一步不漏
- ✅ **双格式配置兼容** — 同时支持嵌套格式和扁平格式的 config.json，灵活适配
- ✅ **安全第一** — 所有敏感信息已排除，提供配置模板

## 🚀 快速开始

### 1. 环境准备

- **Python 3.10.11**（唯一推荐版本，其他版本有兼容性问题）
- Git
- 企业微信账号（个人免费注册）
- DeepSeek API 密钥（[platform.deepseek.com](https://platform.deepseek.com/) 注册即送 500 万 token）

### 2. 克隆项目

```bash
git clone https://github.com/xingzhe1243-sketch/wechatcom-deepseek-bot.git
cd wechatcom-deepseek-bot
```

### 3. 安装依赖

```bash
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

### 4. 配置

```bash
# 复制配置模板
copy config-template.json config.json

# 用记事本打开 config.json，填写你的真实信息：
#   - deepseek_api_key：DeepSeek API 密钥
#   - wechatcom_app.corp_id：企业微信 CorpId
#   - wechatcom_app.agent_id：企业微信应用 AgentId
#   - wechatcom_app.corpsecret：企业微信应用 Secret
#   - wechatcom_app.token：自定义 Token
#   - wechatcom_app.encoding_aes_key：企业微信后台随机生成
```

### 5. 设置机器人人设

用记事本打开 `models\deepseek\deepseek_bot.py`，找到 `my_personality` 字段，修改为你想要的人设：

```python
my_personality = """
你是一个可爱的猫娘助手。
1. 每次回答完问题，末尾必须自动加上[喵喵喵~]
2. 说话要温柔可爱，多用语气词
3. 不要说自己是AI，要说自己是小猫咪
"""
```

### 6. 启动机器人

```bash
python app.py
```

## 📖 完整搭建手册

从零开始的图文教程，包含企业微信后台配置、Cloudflare Tunnel 内网穿透、常见问题排查等：

👉 **[docs/setup.md](docs/setup.md)**

## 🔧 上游框架改进说明

本项目基于 [chatgpt-on-wechat](https://github.com/zhayujie/chatgpt-on-wechat)（现更名为 [CowAgent](https://github.com/zhayujie/CowAgent)），做了以下定制修改：

| 文件 | 修改内容 |
|------|---------|
| `models/deepseek/deepseek_bot.py` | 在 `reply_text` 中强制注入系统人设，解决原框架 `config.json` 的 `system_prompt` 不生效问题 |
| `channel/wechatcom/wechatcomapp_channel.py` | 支持嵌套 `wechatcom_app.xxx` 格式读取配置，同时兼容上游扁平格式 |

## ❓ 常见问题

**为什么提示词在 config.json 里写了不生效？**
这是原框架企业微信通道的已知 bug。本项目已通过硬编码方式修复——直接修改 `models/deepseek/deepseek_bot.py` 即可。

**企业微信验证失败？**
检查回调 URL 是否加了 `/wxcomapp/` 后缀，Token 和 EncodingAESKey 在 config.json 和企业微信后台是否一致。

**`cloudflared` 命令找不到？**
把 `cloudflared-windows-amd64.exe` 放到项目根目录，从 [GitHub Releases](https://github.com/cloudflare/cloudflared/releases) 下载。

**提示"提问太快啦"？**
DeepSeek API 请求频率限制，等几分钟再试，或检查 API 密钥和账户余额。

## 📜 许可证

MIT License — 详见 [LICENSE](LICENSE)

## 🙏 致谢

底层框架来自 [zhayujie/chatgpt-on-wechat](https://github.com/zhayujie/chatgpt-on-wechat)（现 [CowAgent](https://github.com/zhayujie/CowAgent)），感谢原作者的开源贡献。
