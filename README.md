# hello-agent

一个基于 **ReAct 模式**（Thought-Action-Observation）的简易 AI Agent 智能旅行助手。

## 项目简介

本项目实现了一个能够自主推理并使用外部工具的 AI Agent。Agent 通过「思考 → 行动 → 观察」的循环模式，结合大语言模型的推理能力与外部工具，逐步完成复杂的用户请求。

**示例场景**：用户询问"帮我查询杭州天气，并根据天气推荐旅游景点"，Agent 会：
1. 先调用天气查询工具获取实时天气
2. 再根据天气信息调用景点搜索工具
3. 最终整合所有信息，给出完整的旅行建议

## 项目结构

```
hello_agent/
├── agent/                  # Agent 核心模块
│   ├── client.py           # OpenAI 兼容客户端初始化配置
│   └── connect.py          # LLM 客户端封装（OpenAICompatibleClient）
├── tools/                  # 工具模块
│   ├── __init__.py         # 工具注册表（available_tools 字典）
│   ├── weather.py          # 天气查询工具（基于 wttr.in API）
│   └── attraction.py       # 景点推荐工具（基于 Tavily Search API）
├── run.py                  # 主入口，Agent 运行循环
├── system_prompt.md        # Agent 系统提示词
└── README.md
```

## 核心原理

Agent 采用 **ReAct（Reasoning + Acting）** 范式运行，主循环流程如下：

```
用户请求 → [ Thought → Action → Observation ] 循环 → 最终答案
```

1. **构建 Prompt**：将用户请求与历史对话上下文拼接
2. **LLM 思考（Thought）**：模型分析当前情况，规划下一步行动
3. **执行行动（Action）**：调用对应的工具函数
4. **记录观察（Observation）**：将工具返回的结果追加到上下文
5. **循环迭代**：重复步骤 2-4，直到信息充分后调用 `Finish` 输出最终答案

## 内置工具

| 工具名 | 功能 | 数据来源 |
|--------|------|----------|
| `get_weather(city)` | 查询指定城市的实时天气信息 | [wttr.in](https://wttr.in) API |
| `get_attraction(city, weather)` | 根据城市和天气搜索推荐旅游景点 | [Tavily Search](https://tavily.com) API |

## 环境要求

- Python 3.8+
- 依赖包：

```bash
pip install openai requests tavily-python urllib3
```

## 快速开始

### 1. 配置 API 密钥

在 `run.py` 中修改以下配置：

```python
# DeepSeek 大模型
API_KEY = "your-deepseek-api-key"
BASE_URL = "https://api.deepseek.com"
MODEL_ID = "deepseek-v4-flash"

# Tavily 搜索 API
TAVILY_API_KEY = "your-tavily-api-key"
```

- **DeepSeek API Key**：前往 [DeepSeek 开放平台](https://platform.deepseek.com) 获取
- **Tavily API Key**：前往 [Tavily](https://tavily.com) 注册获取

### 2. 修改用户请求（可选）

在 `run.py` 中修改 `user_prompt` 变量来自定义你的问题：

```python
user_prompt = "你好，请帮我查询一下今天杭州的天气，然后根据天气推荐一个合适的旅游景点。"
```

### 3. 运行

```bash
python run.py
```

## 运行示例

```
用户输入: 你好，请帮我查询一下今天杭州的天气，然后根据天气推荐一个合适的旅游景点。
========================================
--- 循环 1 ---

模型输出:
Thought: 用户想知道杭州的天气并获取景点推荐，首先需要查询杭州的实时天气。
Action: get_weather(city="杭州")

正在调用大语言模型...
大语言模型响应成功。
Observation: 杭州当前天气:多云，气温26摄氏度
========================================
--- 循环 2 ---

模型输出:
Thought: 已获取杭州天气为多云、26°C，适合户外游览。接下来根据城市和天气推荐景点。
Action: get_attraction(city="杭州", weather="多云")

正在调用大语言模型...
大语言模型响应成功。
Observation: 杭州多云天气非常适合游览西湖、灵隐寺...
========================================
--- 循环 3 ---

模型输出:
Thought: 已收集到天气和景点信息，可以给出完整回答。
Action: Finish[杭州今日多云，气温26°C，推荐游览西湖和灵隐寺...]

任务完成，最终答案: 杭州今日多云，气温26°C，推荐游览西湖和灵隐寺...
```

## 自定义扩展

如需添加新工具，只需三步：

1. 在 `tools/` 目录下新建 `.py` 文件，编写工具函数
2. 在 `tools/__init__.py` 中导入并注册到 `available_tools` 字典
3. 在 `system_prompt.md` 中添加对应工具的描述

```python
# tools/__init__.py
from tools.your_tool import your_function

available_tools = {
    "get_weather": get_weather,
    "get_attraction": get_attraction,
    "your_tool": your_function,  # 新增工具
}
```
