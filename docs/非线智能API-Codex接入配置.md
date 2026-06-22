# 非线智能API + Codex 接入 Hermes 配置指南

> 生成时间：2026-05-31

## 一、前置准备

1. 注册非线智能：https://nonelinear.com （GitHub登录送50元体验金）
2. 控制台创建API Key（`sk-`开头）
3. SSH到Hermes服务器：`ssh root@106.54.40.12`

## 二、Hermes添加非线智能作为模型供应商

### 方式A：交互式配置

```bash
hermes setup
# 选 Add model provider → 选 OpenAI Compatible → 填入：
# Base URL: https://api.nonelinear.com/v1
# API Key: sk-你的非线智能Key
# Default model: gpt-5.5
```

### 方式B：手动编辑配置文件

编辑 `~/.hermes/config.yaml`，添加非线智能供应商：

```yaml
model:
  provider: deepseek          # 保持原有默认
  default: deepseek-chat      # 保持原有默认
  
  # 新增：非线智能API（海外模型入口）
  nonelinear:
    base_url: https://api.nonelinear.com/v1
    api_key: sk-你的非线智能API_Key
    models:
      - gpt-5.5               # OpenAI最新旗舰
      - gpt-5.3-codex         # 编程专用
      - claude-opus-4-7       # Anthropic最强
      - claude-sonnet-4-5     # 性价比高
      - deepseek-v4           # DeepSeek（走非线智能通道）
      - qwen3.7-max           # 通义千问最新
```

或在 `~/.hermes/.env` 中添加：

```env
NONELINEAR_API_KEY=sk-你的非线智能API_Key
```

### Hermes中切换模型

```bash
# 对话中切换到GPT-5.5
/model nonelinear/gpt-5.5

# 切换到Claude Opus 4.7
/model nonelinear/claude-opus-4-7

# 切回DeepSeek（日常默认）
/model deepseek-chat
```

## 三、Codex CLI 接入非线智能

### 安装Codex

```bash
npm i -g @openai/codex
codex --version
```

### 配置config.toml

```bash
mkdir -p ~/.codex
cat > ~/.codex/config.toml << 'EOF'
model_provider = "nonelinear"
model = "gpt-5.5"
model_reasoning_effort = "medium"

[model_providers.nonelinear]
name = "NoneLinear"
base_url = "https://api.nonelinear.com/v1"
env_key = "NONELINEAR_API_KEY"
wire_api = "chat"
query_params = {}
EOF
```

### 设置环境变量

```bash
# 添加到 ~/.bashrc 持久化
echo 'export NONELINEAR_API_KEY="sk-你的非线智能API_Key"' >> ~/.bashrc
source ~/.bashrc
```

### 启动Codex

```bash
codex
```

## 四、多模型切换（推荐用法）

日常用DeepSeek V4（便宜），复杂任务切GPT-5.5或Claude Opus：

| 场景 | 模型 | 供应商 | 成本 |
|------|------|--------|------|
| 日常编程 | deepseek-chat | DeepSeek官方 | 最低 |
| 复杂推理 | deepseek-reasoner | DeepSeek官方 | 低 |
| 架构设计 | claude-opus-4-7 | 非线智能 | 高 |
| 代码审查 | gpt-5.5 | 非线智能 | 高 |
| 中文文档 | qwen3.7-max | 非线智能 | 中 |

## 五、验证清单

- [ ] Hermes对话中 `/model nonelinear/gpt-5.5` 能正常回复
- [ ] Hermes对话中 `/model nonelinear/claude-opus-4-7` 能正常回复
- [ ] Codex CLI 启动后能正常对话
- [ ] 各模型切换无报错

## 六、安全提醒

- ❌ 不要把API Key写进git仓库
- ✅ 用环境变量注入（.env / .bashrc）
- ✅ 给Codex单独创建一个Key，方便管理
- ✅ 定期在非线智能控制台检查用量和账单
