# 如何在本地使用Ollama

> Ollama的官方Github：https://github.com/ollama/ollama
> ChatOllama的官方GitHub：https://github.com/sugarforever/chat-ollama



## 下载、安装、运行Ollama

以Windows为例，下载并安装好Ollama后既可以通过点击应用图标来打开应用，也可在命令行使用 `ollama serve` 在命令行中开启Ollama。

对于常用的模型，可以使用 `Ollama run <model>` 的指令来下载并运行模型。下面的表格是一些示例的模型、大小以及运行指令。

| Model              | Parameters | Size  | Download                       |
| ------------------ | ---------- | ----- | ------------------------------ |
| Llama 3            | 8B         | 4.7GB | `ollama run llama3`            |
| Llama 3            | 70B        | 40GB  | `ollama run llama3:70b`        |
| Phi-3              | 3.8B       | 2.3GB | `ollama run phi3`              |
| Mistral            | 7B         | 4.1GB | `ollama run mistral`           |
| Neural Chat        | 7B         | 4.1GB | `ollama run neural-chat`       |
| Starling           | 7B         | 4.1GB | `ollama run starling-lm`       |
| Code Llama         | 7B         | 3.8GB | `ollama run codellama`         |
| Llama 2 Uncensored | 7B         | 3.8GB | `ollama run llama2-uncensored` |
| LLaVA              | 7B         | 4.5GB | `ollama run llava`             |
| Gemma              | 2B         | 1.4GB | `ollama run gemma:2b`          |
| Gemma              | 7B         | 4.8GB | `ollama run gemma:7b`          |
| Solar              | 10.7B      | 6.1GB | `ollama run solar`             |

值得注意的是，如果Ollama是通过点击应用图标开启的，那么在命令行中输入 `ollama serve` 会显示端口占用，因为Ollama已经开启了。这时，直接运行大模型即可，参考该 [issue](https://github.com/ollama/ollama/issues/3575)。



## 通过Docker运行ChatOllama

> `ChatOllama` 是一个基于 LLMs（大语言模型）的开源聊天机器人平台，支持多种语言模型，包括：
>
> - Ollama 服务模型
> - OpenAI
> - Azure OpenAI
> - Anthropic
> - Moonshot
> - Gemini
> - Groq
>
> `ChatOllama` 支持多种聊天类型：
>
> - 与 LLMs 免费聊天
> - 基于知识库与 LLMs 聊天
>
> `ChatOllama` 的功能列表：
>
> - Ollama 模型管理
> - 知识库管理
> - 聊天
> - 商业 LLMs API 密钥管理

使用Docker可以实现ChatOllama最简单的运行方式。具体步骤如下：

1. 首先，在本地（任何位置都可以）创建一个文件夹（例如 `ChatOllama-Docker`）；

2. 然后，从ChatOllama的GitHub中复制一份 [docker-compose.yaml](https://github.com/sugarforever/chat-ollama/blob/main/docker-compose.yaml) 配置文件，具体配置如下；

   ```yaml
   version: '3.1'
   services:
     chromadb:
       image: chromadb/chroma
       ports:
         - "8000:8000"
       restart: always
       volumes:
         - chromadb_data:/chroma/.chroma/index
   
     chatollama:
       environment:
         - CHROMADB_URL=http://chromadb:8000
         - DATABASE_URL=file:/app/sqlite/chatollama.sqlite
         - REDIS_HOST=redis
       image: 0001coder/chatollama:latest
       ports:
         - "3000:3000"
       pull_policy: missing
       # pull_policy: always
       restart: always
       volumes:
         - ~/.chatollama:/app/sqlite
   
     redis:
       image: redis:latest
       restart: always
       volumes:
         - redis_data:/data
   
   volumes:
     chromadb_data:
     redis_data:
   ```

3. 接着，打开Docker应用程序，并通过命令行启动Docker compose；

   ```bash
   docker compose up -d
   ```

4. 最后，在浏览器中打开 `http://localhost:3000` 即可使用ChatOllama。并且在 `Setting` 选项卡下的Ollama Server中的Endpoint输入 `http://host.docker.internal:11434`。

   ```bash
   # 浏览器中打开
   http://localhost:3000
   
   # ChatOllama设置中输入
   http://host.docker.internal:11434
   ```

5. 补充说明：

   - 在命令行中可以使用 `docker ps` 查看当前正在运行的容器，正常情况下应该有 `chromadb`、 `chatollama` 和 `redis` 这三个；
   - 使用 `docker compose down` 可以关闭上述所有的、正在运行的容器；
   - 使用知识库时，我们需要一个有效的嵌入模型。在这里可以是Ollama下载的模型或来自第三方服务提供商，例如OpenAI。如果是在本地，推荐使用 `nomic-embed-text` 模型。
