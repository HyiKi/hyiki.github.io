---
title: 本地启动服务支持ChatGPT集成Discord
date:  2023-03-30
category: Original
tags: AIGC
excerpt: AI提升生产力
---

**需求：自定义调教ChatGPT，并集成Discord聊天室server实现Chat Bot，满足日常工作生活需求**

### ChatGPT集成Discord基本步骤

1. 创建Discord Bot账号

要将ChatGPT集成到Discord，您需要创建一个Discord Bot账号。在Discord开发者门户网站（https://discord.com/developers/applications）上创建一个新应用程序，然后将其转换为Bot账号。

2. 获取Bot账号的Token

Bot账号的Token是连接Discord API的关键。获取Token后，您可以使用它来与Discord服务器进行通信。

3. 安装必要的Python库

您需要安装discord.py和openai Python库来与Discord API和ChatGPT进行通信。您可以使用pip命令在命令行中安装这些库。

```
pip install discord.py
pip install openai
```

4. 编写代码

使用Python编写代码，以便Discord Bot可以从用户那里接收消息并将其发送到ChatGPT以获取响应。您可以在OpenAI网站上创建API密钥，以便在代码中调用ChatGPT API。

```python
import discord
import openai
import os

intents = discord.Intents.default()  # 创建一个默认的 Intent 对象
client = discord.Client(intents=intents)  # 指定 Intent 对象

openai.api_key = os.getenv('OPENAI_API_KEY')

@client.event
async def on_ready():
    print('Bot is ready')

@client.event
async def on_message(message):
    if message.author == client.user:
        return

    if message.content.startswith('$hello'):
        response = openai.Completion.create(
            engine="davinci",
            prompt=message.content,
            temperature=0.7,
            max_tokens=100,
            n=1,
            stop=None,
            timeout=15,
            )
        await message.channel.send(response.choices[0].text)

client.run('BOT_TOKEN')
```

5. 运行代码

在命令行中运行代码，启动Discord Bot。

```
python bot.py
```

现在您的ChatGPT已经集成到Discord，并可以通过发送消息与之交互。

6. Discord Bot 加入 Server

   现在转到*OAuth2* (1) 选项卡 - 在这里您可以配置权限并获取指向您的机器人的链接。在*SCOPES*部分，选择***bot*** (2)，在*BOT PERMISSIONS*中，标记您要授予它的权限，在我们的例子中 - 我们只发送消息，因此选择*Send Message* (3)。然后复制自动生成的 Discord 链接 (4)。

### 启动遇到的错误

**aiohttp.client_exceptions.ClientConnectorError: Cannot connect to host discord.com:443 ssl:default** 

1. 排查网络问题 ping discord.com

   发现命令行ping不通，显示连接超时，想到是科学上网的原因，ping 命令是ICMP协议，改用HTTP请求再试一次

   ```sh
   (base) wanghaoxuan@wanghaoxuandeMacBook-Pro ~ % ping discord.com
   PING discord.com (199.96.59.95): 56 data bytes
   Request timeout for icmp_seq 0
   Request timeout for icmp_seq 1
   Request timeout for icmp_seq 2
   ^C
   --- discord.com ping statistics ---
   4 packets transmitted, 0 packets received, 100.0% packet loss
   ```

2. HTTP请求 curl -vv discord.com
   ```sh
   (base) wanghaoxuan@wanghaoxuandeMacBook-Pro ~ % curl -vv discord.com 
   * Could not resolve host: discord.com
   * Closing connection 0
   curl: (6) Could not resolve host: discord.com
   ```

3. 设置proxy代理
   ```sh
   export http_proxy=http://localhost:8001
   export https_proxy=http://localhost:8001
   ```

4. 确认网络连接畅通
   ```sh
   (base) wanghaoxuan@wanghaoxuandeMacBook-Pro ~ % curl -vv https://www.discord.com
   * Uses proxy env variable https_proxy == 'http://localhost:8001'
   *   Trying 127.0.0.1:8001...
   * Connected to localhost (127.0.0.1) port 8001 (#0)
   * allocate connect buffer!
   * Establish HTTP proxy tunnel to www.discord.com:443
   > CONNECT www.discord.com:443 HTTP/1.1
   > Host: www.discord.com:443
   > User-Agent: curl/7.82.0
   > Proxy-Connection: Keep-Alive
   > 
   < HTTP/1.1 200 Connection established
   < 
   * Proxy replied 200 to CONNECT request
   * CONNECT phase completed!
   ```

5. 完成，继续自定义实现Chat Bot功能

### 按照代理的方式优化代码

优化 bot.py 代码，设置代理的方式运行

```python
import discord
import os
import openai

os.environ["http_proxy"] = "http://127.0.0.1:8001"
os.environ["https_proxy"] = "http://127.0.0.1:8001"

TOKEN = ''
OPENAI_API_KEY = ''

intents = discord.Intents.all()

client = discord.Client(intents=intents,proxy="http://127.0.0.1:8001")

openai.api_key = OPENAI_API_KEY
model_engine = "text-davinci-003"

@client.event
async def on_ready():
    print(f'{client.user} has connected to Discord!')

@client.event
async def on_message(message):
    if message.author == client.user:
        return

    print(str(message.content) + ' has received!')

    if message.content.startswith('$chat'):
        user_input = message.content[5:]
        response = openai.Completion.create(
            engine=model_engine,
            prompt=user_input,
            max_tokens=1024,
            n=1,
            stop=None,
            temperature=0.7,
        )
        await message.channel.send(response.choices[0].text)

client.run(TOKEN)
```

执行结果，且能够正常接收信息
```
(base) wanghaoxuan@wanghaoxuandeMacBook-Pro chatgpt % python bot.py
2023-03-23 11:56:14 INFO     discord.client logging in using static token
2023-03-23 11:56:16 INFO     discord.gateway Shard ID None has connected to Gateway (Session ID: 33638fb062b6dfb4853a0828b2db3715).
ChatGPT#2665 has connected to Discord!
$chat can you give me some suggestions on my lunch? has received!
```

### 效果图

![image-20230323115916874](/assets/img/chatbot.png)

![image-20230324194730447](/assets/img/dalle.png)

### 后续玩法

1. 支持上下文
2. 调教Chat Bot https://github.com/PlexPt/awesome-chatgpt-prompts-zh

### 近期AI

1. OpenAI 系列

   https://openai.com/

2. Bing AI 系列 

   https://www.bing.com/new

3. midjourney 生成高精度图片，包含文->图，图->图的能力

   https://www.midjourney.com/

4. google bard

   https://bard.google.com/

5. 文心一言

   https://yiyan.baidu.com/

6. Stable Diffusion 文->图 | 图->图

   https://stablediffusionweb.com/

7. DreamBooth 基础图->文字描述图

   https://dreambooth.github.io/