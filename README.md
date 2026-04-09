# MemOS — AI 长期记忆后端

> 让你的 AI 拥有**跨会话的长期记忆**。基于 Qdrant 向量数据库 + NetworkX 知识图谱 + BM25 混合检索，提供 60+ 个 REST API。

**前端插件仓库** → [my-neuro-plugin-memos](https://github.com/A-night-owl-Rabbit/my-neuro-plugin-memos)

**最近更新（2026-03-29）**：`/search` 修复图谱补全记忆缺少 `created_at` 的问题，并对返回项做 `payload` 时间兜底、可选返回 `id`。详见 [CHANGELOG.md](./CHANGELOG.md)。

---

## 功能概览

| 能力 | 说明 |
|------|------|
| 混合检索 | 向量相似度 + BM25 关键词 + 知识图谱，三路融合 |
| 自动总结 | 对话自动经 LLM 总结后存入向量库，去重合并 |
| 实体提取 | 自动提取人名、地点等实体，构建知识图谱 |
| 图片记忆 | 截图/上传图片，自动描述并向量化 |
| 偏好系统 | 自动学习用户喜好，支持按类别查询 |
| 工具记录 | 记录 AI 调用过的工具，支持回溯 |
| 知识库导入 | 导入网页 URL / txt / pdf / md 文档 |
| 记忆修正 | 修正、补充、删除已有记忆 |
| Web 管理 | Streamlit WebUI，可视化管理所有记忆 |
| 多用户 | 支持多用户隔离（可选） |

---

## 快速部署（5 分钟）

### 前置要求

- **Python 3.10+**（推荐 3.11）
- **pip**（Python 包管理器）
- 约 **2GB 磁盘空间**（Embedding 模型约 1.3GB）

### 第一步：下载代码

```bash
git clone https://github.com/A-night-owl-Rabbit/my-neuro-memos-system.git
cd my-neuro-memos-system
```

### 第二步：安装依赖

```bash
pip install -r requirements.txt
```

> 首次安装 `sentence-transformers` 会自动下载 Embedding 模型（约 1.3GB），需要等待。
>
> 如果网络较慢，可以手动下载模型到本地，然后修改 `config/memos_config.json` 中的 `embedding.model_path` 指向本地路径。

### 第三步：配置 LLM

编辑 `config/memos_config.json`，填入你的 LLM API Key：

```json
{
  "llm": {
    "config": {
      "model": "deepseek-ai/DeepSeek-V3",
      "api_key": "在这里填入你的 API Key",
      "base_url": "https://api.siliconflow.cn/v1"
    }
  }
}
```

> LLM 用于自动总结对话、提取实体，不是对话模型。推荐使用便宜的模型即可。
>
> 支持任何 OpenAI 兼容的 API（SiliconFlow、OpenRouter、本地 Ollama 等）。

### 第四步：启动服务

**Windows 用户**（推荐）：

```
双击 start_memos.bat
```

**命令行启动**：

```bash
python api/memos_api_server_v2.py
```

看到以下输出说明启动成功：

```
================================================================
  MemOS Memory System v2.0
================================================================

  API:  http://127.0.0.1:8003
  Docs: http://127.0.0.1:8003/docs

  Loading model... (15-20 seconds)
```

> 首次启动需要 15-20 秒加载 Embedding 模型，之后每次启动约 5 秒。

### 第五步：验证

打开浏览器访问：**http://127.0.0.1:8003/docs**

看到 FastAPI 的 Swagger 文档页面即表示服务正常运行。

---

## 配合前端插件使用

本仓库是**后端服务**，需要配合前端插件一起使用：

1. 下载前端插件：[my-neuro-plugin-memos](https://github.com/A-night-owl-Rabbit/my-neuro-plugin-memos)
2. 将插件文件放入 `live-2d/plugins/built-in/memos/` 目录
3. 在 `enabled_plugins.json` 中添加 `"built-in/memos"`
4. 先启动本后端服务，再启动主程序

> 前端插件会自动连接 `http://127.0.0.1:8003`，无需额外配置。

---

## 配置说明

配置文件：`config/memos_config.json`

### 存储配置

```json
{
  "storage": {
    "vector": {
      "type": "qdrant",
      "path": "./data/qdrant",
      "collection_name": "memories",
      "vector_size": 1024
    },
    "graph": {
      "type": "networkx",
      "path": "./data/graph_store.json",
      "enabled": true
    }
  }
}
```

### Embedding 模型

```json
{
  "embedding": {
    "model_path": "../full-hub/rag-hub",
    "vector_size": 1024
  }
}
```

> `model_path` 支持 HuggingFace 模型名（自动下载）或本地绝对路径。

### 检索配置

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `search.default_top_k` | 5 | 默认返回记忆条数 |
| `search.similarity_threshold` | 0.6 | 最低相似度阈值 |
| `search.enable_bm25` | true | 启用 BM25 混合检索 |
| `search.bm25_weight` | 0.3 | BM25 权重（向量权重 = 1 - 此值） |
| `search.enable_graph_query` | true | 启用知识图谱增强 |

### LLM 配置（用于记忆处理）

| 配置项 | 说明 |
|--------|------|
| `llm.config.model` | 模型名称 |
| `llm.config.api_key` | API 密钥 |
| `llm.config.base_url` | API 地址 |
| `llm_fallback` | 备用 LLM（主 LLM 失败时自动切换） |

### 功能开关

| 配置项 | 默认值 | 说明 |
|--------|--------|------|
| `entity_extraction.enabled` | true | 自动实体提取 |
| `image.enabled` | true | 图片记忆功能 |
| `image.auto_describe` | true | 图片自动描述 |

---

## 目录结构

```
memos_system/
├── api/                          # API 服务
│   ├── memos_api_server_v2.py    # 主服务（推荐）
│   ├── memos_api_server.py       # 简化版
│   ├── memos_api_server_full.py  # 完整框架版
│   └── routes/                   # 路由模块
│       ├── memory_routes.py      #   记忆相关 API
│       ├── graph_routes.py       #   知识图谱 API
│       ├── cube_routes.py        #   MemCube API
│       ├── chat_routes.py        #   对话 API
│       └── user_routes.py        #   用户管理 API
├── core/                         # 核心模块
│   ├── mos.py                    # Memory OS 核心类
│   ├── graph_manager.py          # 知识图谱管理
│   ├── scheduler.py              # 异步任务调度
│   └── user_manager.py           # 多用户管理
├── storage/                      # 存储层
│   ├── qdrant_client.py          # Qdrant 向量数据库
│   ├── networkx_graph.py         # NetworkX 图存储
│   └── neo4j_client.py           # Neo4j（可选）
├── models/                       # 数据模型
│   ├── memory_item.py            # 记忆条目
│   ├── entity.py                 # 实体
│   ├── relation.py               # 关系
│   └── user.py                   # 用户
├── memories/                     # 记忆模块
│   ├── preference_memory.py      # 偏好记忆
│   ├── tool_memory.py            # 工具使用记忆
│   └── image_memory.py           # 图片记忆
├── memcube/                      # MemCube 容器
│   ├── cube.py
│   └── cube_manager.py
├── utils/                        # 工具类
│   ├── search_utils.py           # BM25 搜索
│   ├── entity_extractor.py       # 实体提取
│   └── document_loader.py        # 文档加载
├── webui/                        # Web 管理界面
│   ├── memos_webui_v3.py         # WebUI 主程序
│   └── lib/                      # 前端静态资源
├── scripts/                      # 工具脚本
│   ├── migrate_memories.py       # 记忆迁移
│   ├── migrate_to_qdrant.py      # 迁移到 Qdrant
│   └── test_memos.py             # 测试脚本
├── config/                       # 配置
│   └── memos_config.json         # 主配置文件
├── data/                         # 数据目录（自动生成）
├── docs/                         # 文档
├── start_memos.bat               # Windows 启动脚本
├── start_memos.ps1               # PowerShell 启动脚本
├── 启动WebUI_v3.bat              # WebUI 启动脚本
└── requirements.txt              # Python 依赖
```

---

## WebUI 管理界面

启动方式：

```
双击 启动WebUI_v3.bat
```

或命令行：

```bash
cd webui
streamlit run memos_webui_v3.py --server.port 8501
```

访问 **http://localhost:8501** 即可在浏览器中：

- 浏览和搜索所有记忆
- 查看知识图谱可视化
- 手动添加/编辑/删除记忆
- 导入文档和网页
- 查看系统状态

---

## API 端点一览

| 方法 | 路径 | 功能 |
|------|------|------|
| POST | `/add` | 添加记忆（支持 LLM 自动总结） |
| POST | `/search` | 混合检索记忆 |
| GET | `/health` | 健康检查 |
| POST | `/memory/feedback` | 修正/补充/删除记忆 |
| POST | `/images/upload` | 上传图片到记忆 |
| POST | `/images/search` | 搜索图片记忆 |
| POST | `/tools/record` | 记录工具使用 |
| GET | `/tools/recent` | 查询最近工具记录 |
| GET | `/preferences/summary` | 偏好摘要 |
| GET | `/preferences` | 偏好列表 |
| POST | `/kb/import` | 导入网页/文档 |

完整 API 文档：启动服务后访问 **http://127.0.0.1:8003/docs**

---

## 端口占用

| 服务 | 端口 | 说明 |
|------|------|------|
| MemOS API | 8003 | 记忆系统后端 |
| WebUI | 8501 | Web 管理界面 |

---

## 常见问题

**Q: 启动报错 `ModuleNotFoundError`？**
A: 运行 `pip install -r requirements.txt` 安装所有依赖。

**Q: 启动报错 `Address already in use`？**
A: 端口 8003 被占用。使用 `start_memos.bat` 会自动清理，或手动关闭占用端口的进程。

**Q: Embedding 模型下载太慢？**
A: 可以手动下载模型，然后修改 `config/memos_config.json` 中的 `embedding.model_path` 指向本地路径。也可以使用国内镜像源。

**Q: 记忆数据存储在哪里？**
A: 向量数据存储在 `data/qdrant/`，知识图谱在 `data/graph_store.json`，图片在 `data/images/`。

**Q: 如何备份记忆？**
A: 备份整个 `data/` 目录即可。

**Q: 支持 macOS / Linux 吗？**
A: 支持。使用命令行 `python api/memos_api_server_v2.py` 启动即可。`.bat` 脚本仅适用于 Windows。

---

## 技术栈

| 组件 | 技术 |
|------|------|
| Web 框架 | FastAPI + Uvicorn |
| 向量数据库 | Qdrant（本地嵌入式） |
| 知识图谱 | NetworkX（轻量 JSON 存储） |
| 混合检索 | 向量相似度 + BM25 + 图谱增强 |
| Embedding | SentenceTransformer（本地模型） |
| 记忆处理 | LLM 自动总结、去重、合并、分类 |
| Web 管理 | Streamlit |

---

## 想邀请你，做这只小牛的“云饲养员”

做这个桌宠的初衷，其实是因为自己一个人工作学习的时候，总觉得屏幕里空落落的。看到大家都在使用，我就觉得熬夜写代码、调教AI的日子都亮闪闪的。🌟

不过，肥牛现在还在长身体（其实是我想给它做更多有趣的插件），养一只数字小牛其实也挺“费草”的哈哈。🌱

如果你在这只小肥牛这里获得过哪怕一秒钟的治愈，或者觉得它算个合格的桌面搭子，要不要考虑成为它的“云饲养员”呀？

你的每一次充电，都不是在打赏我，而是在给这只肥牛注入一点点魔法值。让它能变得更聪明、更通人性、能听懂你更多的碎碎念。

不用有压力哦！你愿意打开它，就是对我最大的鼓励啦。如果刚好有余力，就请肥牛喝瓶快乐水叭，它会记住你的味道的！🥤❤️

爱发电 https://ifdian.net/a/0923A

## 许可

MIT License
