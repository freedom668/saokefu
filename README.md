# 智扫通机器人智能客服

基于 LangChain/LangGraph ReAct Agent + 通义千问大模型的智能客服系统，集成 RAG 知识库检索，提供智能问答与用户使用报告生成服务。

## 技术栈

| 组件 | 技术选型 |
|------|----------|
| Web UI | Streamlit |
| Agent 框架 | LangChain + LangGraph (ReAct) |
| 大模型 | 通义千问 qwen3-max (阿里云 DashScope) |
| Embedding | text-embedding-v4 (阿里云 DashScope) |
| 向量数据库 | Chroma |
| 文档加载 | PyPDF + TextLoader |

## 项目结构

```
├── app.py                  # Streamlit Web 入口
├── agent/
│   ├── react_agennt.py     # ReAct Agent 核心实现
│   └── tools/
│       ├── agent_tools.py  # 工具定义 (RAG检索、天气、用户数据等)
│       └── middleware.py   # 中间件 (日志、动态提示词切换)
├── model/
│   └── factory.py          # 模型工厂 (ChatModel + Embedding)
├── rag/
│   ├── rag_service.py      # RAG 检索服务
│   └── vector_store.py     # Chroma 向量库管理
├── config/
│   ├── rag.yml             # 模型配置
│   ├── chroma.yml          # 向量库配置
│   ├── agent.yml           # Agent 配置
│   └── prompts.yml         # 提示词路径配置
├── prompts/
│   ├── main_prompt.txt     # 主系统提示词
│   ├── rag_summarize.txt   # RAG 总结提示词
│   └── report_prompt.txt   # 报告生成提示词
├── data/
│   ├── *.txt / *.pdf       # 知识库文档
│   └── external/           # 外部数据源
│       └── records.csv     # 用户使用记录
├── utils/
│   ├── config_handler.py   # YAML 配置加载
│   ├── prompt_loader.py    # 提示词加载
│   ├── file_handler.py     # 文件处理 (MD5、PDF/TXT加载)
│   ├── path_tool.py        # 路径工具
│   └── logger_handler.py   # 日志工具
├── chroma_db/              # Chroma 持久化目录
└── logs/                   # 日志目录
```

## 快速开始

### 环境要求

- Python >= 3.10
- 阿里云 DashScope API Key（[申请地址](https://dashscope.console.aliyun.com/)）

### 安装

```bash
# 进入项目目录
cd Agent_app

# 创建虚拟环境（推荐）
python -m venv venv

# 激活虚拟环境
# Windows
venv\Scripts\activate
# Linux/macOS
source venv/bin/activate

# 安装依赖
pip install -r requirements.txt
```

### 配置 API Key

复制 `.env.example` 为 `.env`，填入真实 API Key：

```bash
cp .env.example .env
# 编辑 .env，替换为你的真实 API Key
```

加载环境变量：

**Windows PowerShell**
```powershell
get-content .env | foreach { $name, $value = $_.split('='); set-item env:$name $value }
```

**Windows CMD**
```cmd
for /f "tokens=*" %i in (.env) do set %i
```

**Linux/macOS**
```bash
export $(cat .env | xargs)
```

> `.env` 已在 `.gitignore` 中排除，不会被提交到 Git。API Key 推荐从 [阿里云 DashScope 控制台](https://dashscope.console.aliyun.com/) 获取。

### 初始化知识库

将产品文档（PDF/TXT）放入 `data/` 目录，然后运行向量化脚本：

```bash
cd rag
python vector_store.py
```

系统会通过 MD5 去重，已导入的文档不会重复处理。

### 启动应用

```bash
streamlit run app.py
```

浏览器访问 `http://localhost:8501` 即可使用。

## 功能说明

### 智能问答

基于 RAG 架构，从知识库中检索相关文档片段，结合大模型生成准确回答。覆盖以下知识域：

- 扫地机器人选购指南
- 使用说明与操作指南
- 故障排除与维修方法
- 维护保养建议

### Agent 工具

ReAct Agent 可自主调用以下工具完成复杂任务：

| 工具 | 功能 |
|------|------|
| `rag_summarize` | 向量库语义检索，获取产品知识 |
| `get_weather` | 获取指定城市天气信息 |
| `get_user_location` | 获取当前用户所在城市 |
| `get_user_id` | 获取当前用户 ID |
| `get_current_month` | 获取当前月份 |
| `fetch_external_data` | 从外部系统查询用户历史使用记录 |
| `fill_context_for_report` | 触发报告生成模式（动态切换提示词） |

### 使用报告

当用户请求生成使用报告时，Agent 的工作流程：

1. 获取用户 ID、所在城市等上下文信息
2. 调用 `fetch_external_data` 拉取该用户指定月份的使用记录
3. 调用 `fill_context_for_report` 触发中间件切换至报告专用提示词
4. 综合用户数据与知识库信息，生成结构化个性化报告

## 配置说明

### 模型配置 (`config/rag.yml`)

```yaml
chat_model_name: qwen3-max              # 对话模型
embedding_model_name: text-embedding-v4 # 向量化模型
```

### 向量库配置 (`config/chroma.yml`)

```yaml
collection_name: agent                  # Chroma 集合名称
persist_directory: chroma_db            # 向量库持久化路径
k: 3                                    # 检索返回条数
data_path: data                         # 知识库文件目录
chunk_size: 200                         # 文档分块大小
chunk_overlap: 20                       # 分块重叠长度
separators: ["\n\n", "\n", ".", "!", "?", "。", "！", "？", " ", ""]
```

### Agent 配置 (`config/agent.yml`)

```yaml
external_data_path: data/external/records.csv  # 外部用户数据路径
```

### 提示词配置 (`config/prompts.yml`)

```yaml
main_prompt_path: prompts/main_prompt.txt                 # 主对话提示词
rag_summarise_prompt_path: prompts/rag_summarize.txt       # RAG 总结提示词
report_prompt_path: prompts/report_prompt.txt              # 报告生成提示词
```

## 数据格式

### 外部用户数据 (`data/external/records.csv`)

CSV 格式，包含字段：用户ID、特征、效率、耗材、对比、月份。Agent 在生成报告时会按用户 ID 和月份检索对应记录。

## 安全提示

- **API Key 保护**：务必使用 `.env` 文件存放 `DASHSCOPE_API_KEY`，`.env` 已加入 `.gitignore`，不会被意外提交
- **日志文件**：`logs/*.log` 已排除，运行产生的日志不会进入版本控制
- **向量库数据**：`chroma_db/` 目录已排除，本地向量索引不会被上传
- **检查历史**：如有需要，可使用 `git log --all --full-history -- <path>` 确认某个文件是否存在于 Git 历史中
