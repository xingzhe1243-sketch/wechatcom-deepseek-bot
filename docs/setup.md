# 零基础小白：企业微信+DeepSeek机器人 从0到1原理+实操完整手册

> 基于 [CowAgent](https://github.com/zhayujie/CowAgent)（原 chatgpt-on-wechat）框架  
> 适用版本：Python 3.10.11 + DeepSeek API

---

## 一、前置准备：安装经过验证的稳定版本软件

### 1.1 为什么绝对不能用Python最新版？

CowAgent 项目的第三方插件（如企业微信消息处理库、AI接口调用库）更新速度严重滞后于 Python 最新版。使用 Python 3.12+ 会出现大量 `ModuleNotFoundError`（模块找不到）错误，这是小白 90% 会踩的第一个坑。

**唯一推荐版本：Python 3.10.11**（所有插件 100% 兼容，无任何兼容性问题）

### 1.2 安装 Python 3.10.11（一步都不能错）

1. 直接下载官方安装包：  
   https://www.python.org/ftp/python/3.10.11/python-3.10.11-amd64.exe
2. 运行安装程序，**必须勾选最下面的 "Add Python 3.10 to PATH"**（这是整个安装过程中最关键的一步！不勾选后面所有命令都会报错）
3. 点击 "Customize installation"（自定义安装）
4. 所有选项保持默认，一直点击 "Next"
5. 在最后一页，**勾选 "Install for all users"**，然后点击 "Install"
6. 安装完成后，点击 "Close"

**验证安装成功**：
- 按下 `Win + R` 组合键，输入 `cmd`，回车打开命令提示符（黑窗口）
- 输入 `python --version`，回车
- 必须精确显示 `Python 3.10.11`，否则就是安装失败，需要卸载重装

### 1.3 安装 Git（下载项目代码专用）

**原理讲解**：Git 是一个代码版本管理工具，我们用它来把网上的机器人项目代码完整地下载到本地电脑。

1. 下载官方安装包：  
   https://github.com/git-for-windows/git/releases/download/v2.45.1.windows.1/Git-2.45.1-64-bit.exe
2. 运行安装程序，一直点击 "Next"，所有选项保持默认即可
3. 安装完成后，**重新打开一个 cmd 窗口**（必须重新打开，否则环境变量不生效）
4. 输入 `git --version`，回车
5. 显示类似 `git version 2.45.1.windows.1` 说明安装成功

---

## 二、下载并初始化机器人项目

### 2.1 为什么要用 CowAgent（原 chatgpt-on-wechat）项目？

这是一个已经写好的成熟机器人框架，它帮我们处理了所有复杂的底层逻辑：
- 接收企业微信发来的消息
- 管理聊天上下文和用户记忆
- 把消息格式化后转发给 DeepSeek AI
- 把 AI 的回复再发回给企业微信
- 处理各种异常情况和错误

我们不需要自己写几千行代码，只需要配置几个参数就能拥有一个功能完整的机器人。

### 2.2 下载项目代码

打开 cmd 窗口，输入下面的命令，回车：

```bash
git clone https://github.com/zhayujie/CowAgent.git
```

**解释**：这个命令会在你的电脑上创建一个名为 `CowAgent` 的文件夹，里面包含了机器人的所有源代码。

### 2.3 进入项目文件夹

```bash
cd CowAgent
```

**解释**：`cd` 是"切换目录"的命令，我们必须进入这个文件夹才能执行后续的所有操作。执行成功后，你会看到 cmd 窗口前面的路径变成了 `C:\Users\你的用户名\CowAgent>`。

### 2.4 安装项目依赖（非常重要！）

**原理讲解**：机器人运行需要很多别人写好的代码库（叫做"依赖"），比如发送 HTTP 请求、处理 JSON 数据的库。`pip` 是 Python 自带的依赖安装工具。

在项目文件夹里输入：

```bash
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

**解释**：
- `-r requirements.txt`：告诉 pip 安装这个文件里列出的所有依赖
- `-i https://pypi.tuna.tsinghua.edu.cn/simple`：使用清华镜像源，下载速度会比官方源快 10 倍以上

等待所有依赖安装完成，这一步可能需要 3-5 分钟，取决于你的网络速度。

---

## 三、配置文件 config.json 详解（最容易出错的部分）

### 3.1 什么是 config.json？为什么要改它？

config.json 是机器人的"大脑设置文件"，就像你手机里的"设置"APP一样。机器人启动时会先读取这个文件，知道：
- 我要连接哪个 AI 模型（DeepSeek）
- 我要用什么方式接收消息（企业微信）
- 我的 API 密钥是什么
- 我要在哪个端口运行

### 3.2 JSON 格式铁律（违反任何一条，机器人都会启动失败）

JSON 的语法非常严格，写错一个符号都会导致配置失效：
1. 所有的键名（冒号前面的文字）必须用**双引号**括起来
2. 键名里**绝对不能有空格**，必须用下划线 `_` 连接
3. 字符串值（冒号后面的文字）也必须用**双引号**括起来
4. 最后一个键值对后面**不能有逗号**

❌ 错误示例：
```json
"channel type": "wechatcom app",
```
> 键名有空格，错误！

✅ 正确示例：
```json
"channel_type": "wechatcom_app",
```
> 键名用下划线，正确！

### 3.3 一步步创建并填写 config.json

1. 打开文件资源管理器，进入 `C:\Users\你的用户名\CowAgent` 文件夹
2. 找到 `config-template.json` 文件，右键点击它，选择"复制"
3. 在空白处右键点击，选择"粘贴"
4. 把复制出来的 `config-template - 副本.json` 重命名为 `config.json`
5. 右键点击 `config.json`，选择"打开方式" → "记事本"
6. 把里面的内容替换为你的真实配置

### 3.4 分支说明：config.json 格式兼容性

本项目 fork 自 CowAgent，对 `channel/wechatcom/wechatcomapp_channel.py` 做了格式兼容改进，同时支持两种配置格式：

**方式一：嵌套格式（本手册推荐）**

```json
{
  "channel_type": "wechatcom_app",
  "model": "deepseek-chat",
  "deepseek_api_key": "sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "deepseek_api_base": "https://api.deepseek.com/v1",
  "system_prompt": "每次回答加[喵喵喵~]",
  "port": 9999,
  "wechatcom_app": {
    "corp_id": "wwxxxxxxxxxxxxxx",
    "agent_id": 1000002,
    "corpsecret": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "token": "abc123def456",
    "encoding_aes_key": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
  }
}
```

**方式二：扁平格式（上游原版）**

```json
{
  "channel_type": "wechatcom_app",
  "wechatcom_corp_id": "CORPID",
  "wechatcomapp_token": "TOKEN",
  "wechatcomapp_port": 9999,
  "wechatcomapp_secret": "SECRET",
  "wechatcomapp_agent_id": 1000002,
  "wechatcomapp_aes_key": "AESKEY"
}
```

> 两种格式任选其一即可，代码会自动识别。

### 3.5 逐个填写每个配置项

| 配置项 | 是什么 | 怎么填 |
|--------|--------|--------|
| `channel_type` | 消息通道类型 | 固定填 `"wechatcom_app"`，表示使用企业微信应用通道 |
| `model` | 使用的AI模型 | 固定填 `"deepseek-chat"`，这是 DeepSeek 官方支持的稳定模型 |
| `deepseek_api_key` | DeepSeek的API密钥 | 下一节教你怎么获取 |
| `deepseek_api_base` | DeepSeek的API地址 | 固定填 `"https://api.deepseek.com/v1"` |
| `system_prompt` | 机器人的人设 | **重要提醒：这个字段对企业微信通道不生效！** 请参考第七章硬编码人设 |
| `port` | 机器人运行的端口 | 固定填 `9999`，后面 Cloudflare Tunnel 会用到 |
| `corp_id` | 企业微信的企业ID | 从企业微信后台获取 |
| `agent_id` | 企业微信应用的ID | 从企业微信后台获取 |
| `corpsecret` | 企业微信应用的密钥 | 从企业微信后台获取 |
| `token` | 消息验证令牌 | 自己随便写一串英文和数字，比如 `"abc123def456"` |
| `encoding_aes_key` | 消息加密密钥 | 从企业微信后台随机生成 |

### 3.6 获取 DeepSeek API 密钥

1. 打开浏览器，访问 DeepSeek 开放平台：https://platform.deepseek.com/
2. 用手机号注册并登录账号
3. 点击左侧菜单栏的"API密钥"
4. 点击右上角的"创建新的密钥"
5. 输入一个名称（比如"我的机器人"），点击"创建"
6. 复制生成的密钥（看起来像 `sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`）
7. 粘贴到 `config.json` 里的 `deepseek_api_key` 双引号中间

---

## 四、企业微信后台配置详解

### 4.1 为什么必须用企业微信？

普通微信没有公开的机器人 API，任何第三方微信机器人都是基于逆向工程的破解版，有极高的封号风险（轻则限制登录，重则永久封号）。

而企业微信提供了官方的"接收消息API"，完全合法合规，永远不会封号。并且企业微信和普通微信是互通的，你可以在企业微信里加普通微信好友，别人给你发消息，机器人也能自动回复。

### 4.2 注册企业微信（个人也能免费注册）

1. 打开浏览器，访问企业微信官网：https://work.weixin.qq.com/
2. 点击右上角的"立即注册"
3. 选择"企业/组织"注册
4. 填写企业名称（随便写，比如"个人工作室"）、行业、规模
5. 填写管理员姓名和手机号，完成短信验证
6. 点击"注册"，完成企业创建

### 4.3 创建自建应用

1. 登录企业微信管理后台：https://work.weixin.qq.com/wework_admin/frame
2. 点击左侧菜单栏的"应用管理" → "应用"
3. 点击"自建"下面的"创建应用"
4. 上传应用头像（随便找一张图片）
5. 填写应用名称（比如"我的AI助手"）和应用介绍
6. 可见范围选择你自己（只有你能看到这个应用）
7. 点击"创建应用"

### 4.4 获取企业微信的三个关键信息

创建好应用后，你会进入应用详情页面：

1. **AgentId**：就在应用名称下面，是一串数字（比如 `1000002`）
   - 复制下来，粘贴到 `config.json` 里的 `agent_id` 后面（注意不要加引号）
2. **Secret**：点击"Secret"后面的"查看"
   - 点击"发送"，会把 Secret 发送到你的企业微信客户端上
   - 打开企业微信，复制收到的 Secret
   - 粘贴到 `config.json` 里的 `corpsecret` 双引号中间
3. **CorpId**：点击页面右上角你的企业名称
   - 点击"企业信息"
   - 拉到最下面，找到"企业ID"
   - 复制下来，粘贴到 `config.json` 里的 `corp_id` 双引号中间

### 4.5 配置接收消息服务器（先暂停，下一章完成后再回来）

**原理讲解**：企业微信需要一个公网地址来给你的机器人发消息。但你的电脑是在家里的局域网里，外网访问不到。所以我们需要用"内网穿透"工具，把你的本地电脑暴露到公网上。

我们用的是 Cloudflare Tunnel，这是一个免费、稳定、不需要公网 IP、不需要配置路由器的内网穿透工具。

---

## 五、Cloudflare Tunnel 配置详解

### 5.1 什么是 Cloudflare Tunnel？为什么要用它？

Cloudflare Tunnel 会在你的电脑和 Cloudflare 的全球服务器之间建立一个加密的隧道。所有发往 Cloudflare 公网地址的请求，都会通过这个隧道转发到你电脑的 9999 端口（也就是机器人运行的端口）。

这样企业微信就能通过公网地址，把消息发送到你本地的机器人上了。

### 5.2 下载并运行 Cloudflare Tunnel

1. 打开浏览器，下载最新版的 cloudflared：  
   https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-windows-amd64.exe
2. 把下载好的文件，**复制粘贴到你的项目文件夹里**（`C:\Users\你的用户名\CowAgent`）
   - 这一步非常重要！如果不放到项目文件夹里，cmd 窗口会找不到这个程序
3. **打开一个新的 cmd 窗口**（必须是新的！后面我们需要两个窗口同时运行）
4. 输入 `cd CowAgent`，回车，进入项目文件夹
5. 输入下面的命令，回车：

```bash
cloudflared-windows-amd64.exe tunnel --url http://localhost:9999
```

**解释**：这个命令会创建一个临时隧道，把所有发往 Cloudflare 公网地址的请求，转发到你本地电脑的 9999 端口。

6. 运行成功后，你会看到类似下面的输出：

```
2026-05-11T14:27:18Z INF Your quick Tunnel has been created! Visit it at:
2026-05-11T14:27:18Z INF |  https://erp-sells-trance-submission.trycloudflare.com  |
```

把这个 `https://xxx.trycloudflare.com` 地址复制下来，这就是你的公网地址。

### 5.3 回到企业微信后台完成配置

1. 回到企业微信应用详情页面
2. 找到"功能"模块里的"接收消息"，点击"设置API接收"
3. 在"URL（服务器地址）"里，粘贴你刚才复制的公网地址
   - **必须在后面加上 `/wxcomapp/` 后缀！** 这是 90% 的人会犯的错误
   - 完整地址应该是：`https://erp-sells-trance-submission.trycloudflare.com/wxcomapp/`
4. 在"Token"里，填写你在 `config.json` 里写的那个 token（必须完全一样）
5. 在"EncodingAESKey"里，点击"随机生成"
   - 复制生成的密钥
   - 粘贴到 `config.json` 里的 `encoding_aes_key` 双引号中间
6. 消息加解密模式选择"兼容模式"
7. 点击"保存"
   - 如果显示"验证成功"，说明所有配置都正确
   - 如果显示"验证失败"，检查 URL 后缀、Token、EncodingAESKey 是否一致

---

## 六、启动机器人并测试

### 6.1 启动机器人

1. 回到第一个 cmd 窗口（已经在项目文件夹里了）
2. 输入下面的命令，回车：

```bash
python app.py
```

3. 如果没有报错，说明机器人启动成功了

现在你有两个 cmd 窗口同时运行：
- **窗口1**：运行机器人程序，处理消息
- **窗口2**：运行 Cloudflare Tunnel，转发公网请求

**重要提醒：两个窗口都不能关！** 关闭任何一个，机器人都会停止工作。也不要最小化到托盘，保持窗口打开状态。

### 6.2 测试机器人

1. 打开企业微信客户端
2. 点击左侧的"工作台"
3. 找到你创建的应用，点击进入聊天界面
4. 发送一条消息，比如"你好"
5. 你应该会收到机器人的回复，说明机器人已经正常工作了

---

## 七、终极解决：提示词不生效问题（核心中的核心！）

### 7.1 为什么 config.json 里的 system_prompt 不生效？

这是 CowAgent 项目的一个**已知且长期存在的 bug**：企业微信通道的代码没有正确读取 config.json 里的 system_prompt 字段。

无论你在 config.json 里写什么人设，机器人都会用默认的"我是DeepSeek，一个乐于助人的AI助手"来回复你。这就是你之前所有尝试都失败的根本原因。

### 7.2 终极解决方案：硬编码人设

我们直接修改机器人的源代码，把人设"焊死"在里面。这样不管配置文件怎么写，机器人都会严格按照你的人设回复。

**为什么这个方法100%生效？**
因为我们直接在机器人调用 DeepSeek API 之前，把你的人设强制插入到请求里。DeepSeek 收到的每一个请求，都会先看到你的人设规则。

### 7.3 一步步修改代码

1. 打开文件资源管理器，进入项目文件夹里的 `models/deepseek` 目录：  
   `C:\Users\你的用户名\CowAgent\models\deepseek`
2. 找到 `deepseek_bot.py` 文件，右键点击它，选择"打开方式" → "记事本"
3. 往下翻，找到 `reply_text` 函数（大概在第 155 行左右）
   - 你会看到这样的代码：

```python
def reply_text(self, session, args=None, retry_count: int = 0) -> dict:
    try:
        headers = self._build_headers()
        body = dict(args) if args else dict(self.args)
        body["messages"] = session.messages
```

4. 找到这两行关键代码：

```python
body = dict(args) if args else dict(self.args)
body["messages"] = session.messages
```

5. 把这两行代码，**完整替换成下面这段**：

```python
body = dict(args) if args else dict(self.args)

# ====================== 在这里修改你的机器人人设 ======================
# 所有的规则都写在下面的三个双引号里，支持换行（直接按回车）
my_personality = """
你是一个可爱的猫娘助手。
1. 每次回答完问题，末尾必须自动加上[喵喵喵~]
2. 说话要温柔可爱，多用语气词
3. 不要说自己是AI，要说自己是小猫咪
"""
# ===================================================================

# 创建固定的系统人设消息
fixed_system = {"role": "system", "content": my_personality}

# 构建新的消息列表：先放我们的固定人设，再放原来的聊天记录
new_messages = [fixed_system]
for message in session.messages:
    # 跳过原来的系统消息，只用我们的固定人设
    if message.get("role") != "system":
        new_messages.append(message)

# 把新的消息列表传给API
body["messages"] = new_messages
```

6. 保存文件（按 `Ctrl + S`）
7. 回到运行机器人的 cmd 窗口
8. 按下 `Ctrl + C` 停止机器人
9. 重新输入 `python app.py`，回车，重启机器人

### 7.4 测试效果

现在再去企业微信里给机器人发消息，你会发现它已经完全按照你的人设回复了！

**以后想修改人设**，只需要：
1. 打开 `models/deepseek/deepseek_bot.py` 文件
2. 修改 `my_personality` 三个双引号里的内容
3. 保存文件
4. 重启机器人

---

## 八、常见问题终极排查手册

### 问题1：'python' 不是内部或外部命令

**原因**：安装 Python 时没有勾选"Add Python to PATH"

**解决方法**：
1. 卸载 Python
2. 重新安装，**必须勾选最下面的"Add Python 3.10 to PATH"**

### 问题2：'cloudflared-windows-amd64.exe' 不是内部或外部命令

**原因**：没有把 cloudflared.exe 放到项目文件夹里

**解决方法**：
1. 确认 `cloudflared-windows-amd64.exe` 已经在 `CowAgent` 文件夹里
2. 确认 cmd 窗口已经进入了 `CowAgent` 文件夹

### 问题3：企业微信验证失败

**可能的原因和解决方法**：
1. ❌ URL 没有加 `/wxcomapp/` 后缀 → 必须加上！
2. ❌ Token 或 EncodingAESKey 不一致 → 检查企业微信后台和 config.json 里的是否完全一样
3. ❌ 机器人或 Cloudflare Tunnel 没有运行 → 确认两个窗口都在运行，没有报错
4. ❌ 端口不一致 → 确认 config.json 里的 port 是 9999，Cloudflare Tunnel 命令里也是 9999

### 问题4：机器人回复"提问太快啦，请休息一下再问我吧"

**原因**：DeepSeek API 请求过于频繁，或者 API 密钥有问题

**解决方法**：
1. 等待 5 分钟再试
2. 检查你的 DeepSeek API 密钥是否正确
3. 检查你的 DeepSeek 账户是否有余额（新用户有 500 万免费 token 额度）

### 问题5：提示词还是不生效

**原因**：没有正确修改代码，或者没有重启机器人

**解决方法**：
1. 仔细核对修改的代码是否和教程里给的完全一样
2. 确认已经保存了文件（按 Ctrl+S）
3. 确认已经按下 Ctrl+C 停止了机器人，然后重新运行了 python app.py
4. 确认修改的文件路径是 `models/deepseek/deepseek_bot.py`（注意多了一层 deepseek 目录）

---

## 九、日常使用说明

1. **两个窗口都不能关**：运行机器人的窗口和运行 Cloudflare Tunnel 的窗口都不能关闭，也不能最小化到托盘。
2. **每次重启电脑都需要重新启动**：
   - 打开 cmd 窗口1：`cd CowAgent` → `python app.py`
   - 打开 cmd 窗口2：`cd CowAgent` → `cloudflared-windows-amd64.exe tunnel --url http://localhost:9999`
   - 复制新的公网地址，去企业微信后台更新回调 URL（每次重启 cloudflared 地址会变）
3. **修改人设**：直接修改 `models/deepseek/deepseek_bot.py` 里的 `my_personality` 字段，然后重启机器人。
4. **添加更多功能**：这个项目支持很多插件，比如画图、语音、天气查询等，可以去 [CowAgent 官网](https://cowagent.ai/) 查看文档。

---

## 快速检查清单（确保每一步都做对）

- ✅ Python 3.10.11 安装成功，并且添加到了 PATH
- ✅ Git 安装成功
- ✅ 项目代码已经用 git clone 下载到本地
- ✅ 项目依赖已经用 pip install -r requirements.txt 安装完成
- ✅ config.json 格式正确，键名没有空格
- ✅ DeepSeek API 密钥已经正确填写
- ✅ 企业微信应用已经创建，AgentId、Secret、CorpId 都正确填写
- ✅ cloudflared.exe 已经放到项目文件夹里
- ✅ Cloudflare Tunnel 已经运行，生成了公网地址
- ✅ 企业微信回调 URL 已经填写，并且加了 `/wxcomapp/` 后缀
- ✅ Token 和 EncodingAESKey 在企业微信后台和 config.json 里一致
- ✅ 已经修改了 `models/deepseek/deepseek_bot.py` 文件，强制注入了人设
- ✅ 机器人和 Cloudflare Tunnel 两个窗口都在运行，没有报错
