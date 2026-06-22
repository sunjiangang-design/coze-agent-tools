# Hermes Agent + DeepSeek V4 + 飞书 完整部署清单

> 适用环境：Windows 10/11 | 更新日期：2026-05-06

---

## 一、前置条件

- [ ] Windows 10 2004+ 或 Windows 11
- [ ] 8GB+ 内存，20GB+ 磁盘空间
- [ ] 不需要GPU（AI计算全在云端）
- [ ] DeepSeek API Key（platform.deepseek.com 充值获取）
- [ ] 飞书管理员权限（用于创建机器人应用）

---

## 二、安装 Hermes Agent

### 方案A：PowerShell 一键安装（快速体验）

```powershell
# 管理员 PowerShell
irm https://res1.hermesagent.org.cn/install.ps1 | iex
```

### 方案B：WSL2 安装（长期推荐 ⭐）

```powershell
# 1. 安装WSL2
wsl --install
# 重启电脑

# 2. 进入Ubuntu
wsl

# 3. 更新依赖
sudo apt update && sudo apt install -y ripgrep ffmpeg

# 4. 一键安装Hermes（国内镜像加速）
curl -fsSL https://res1.hermesagent.org.cn/install.sh | bash
```

### 验证安装

```bash
hermes --version
hermes doctor
```

---

## 三、配置 DeepSeek V4

### 交互式配置

```bash
hermes setup
# 选 Quick setup → 模型供应商选 DeepSeek → 输入API Key → 模型选 deepseek-chat
```

### 手动编辑配置文件

编辑 `~/.hermes/config.yaml`：

```yaml
model:
  provider: deepseek
  default: deepseek-chat
  deepseek:
    base_url: https://api.deepseek.com/v1
    api_key: sk-xxxxxxxxxxxxxxxx
```

或编辑 `~/.hermes/.env`：

```env
DEEPSEEK_API_KEY=sk-xxxxxxxxxxxxxxxx
```

### 模型说明

| 模型ID | 实际模型 | 价格 | 适用场景 |
|---|---|---|---|
| deepseek-chat | V4 Flash（默认） | 输入0.1/输出2元/百万token | 日常对话、简单编程 |
| deepseek-reasoner | V4 Pro | 输入2/输出8元/百万token(2.5折) | 复杂推理、深度分析 |

> V4 Pro 2.5折优惠截止 2026-05-31 23:59

### 切换模型

```bash
hermes model          # 交互式切换
# 对话中切换
/model deepseek-reasoner
```

### 验证对话

```bash
hermes
# 输入任意问题，确认能正常回复
/exit
```

---

## 四、接入飞书

### 4.1 创建飞书机器人应用

1. 打开 https://open.feishu.cn ，管理员登录
2. 创建应用 → 企业自建应用 → 填名称（如"Hermes AI 助手"）
3. 添加机器人能力：应用详情 → 添加应用能力 → 选"机器人"
4. 配置权限（权限管理）：

| 权限 | 用途 |
|---|---|
| `im:message` | 收发消息 |
| `im:message.group_at_msg` | 接收群@消息 |
| `im:message.p2p_msg` | 接收私聊消息 |
| `contact:user.id:readonly` | 读取用户ID |

5. 记录凭证：凭证与基础信息页 → 记下 **App ID** 和 **App Secret**
6. 配置事件订阅：
   - 添加事件：`im.message.receive_v1`
   - （可选）添加：`card.action.trigger`（交互卡片按钮）
7. 启用交互式卡片（可选）：应用功能 → 机器人 → 开启
8. 发布应用：版本管理与发布 → 创建版本 → 提交审核 → 管理员审批

### 4.2 配置 Hermes Gateway

**交互式配置（推荐）：**

```bash
hermes gateway setup
# 选 Feishu / Lark
# 填入 App ID
# 填入 App Secret
# 来源：feishu（国内版）
# 连接方式：websocket（无需公网IP，直接回车）
# 允许的用户ID：ou_162d8a7a9558908aca3608a0c19f6ed7
```

**手动编辑 `~/.hermes/.env`：**

```env
FEISHU_APP_ID=cli_xxxxxxxxxx
FEISHU_APP_SECRET=xxxxxxxxxxxxxxx
FEISHU_DOMAIN=feishu
FEISHU_CONNECTION_MODE=websocket
FEISHU_ALLOWED_USERS=ou_162d8a7a9558908aca3608a0c19f6ed7
FEISHU_GROUP_POLICY=open
```

### 4.3 启动网关

```bash
hermes gateway
# 看到 "Feishu connected" 即成功
```

### 4.4 测试

飞书中搜索机器人名称 → 发消息 → 确认回复

---

## 五、Windows 常见坑修复

### 坑1：缺少 lark-oapi 依赖

```powershell
# 找到hermes venv路径
Get-Command hermes | Select-Object -ExpandProperty Source
# 安装依赖（替换<用户名>）
uv pip install lark-oapi --python "C:\Users\<用户名>\AppData\Local\hermes\hermes-agent\venv\Scripts\python.exe"
```

### 坑2：群消息被静默丢弃

```powershell
Add-Content "$env:LOCALAPPDATA\hermes\.env" "`nFEISHU_GROUP_POLICY=open" -Encoding UTF8
```

### 坑3：日志中文乱码

```powershell
$env:PYTHONUTF8 = "1"
hermes gateway
```

### 坑4：OSError WinError 进程检测异常

```powershell
# Patch status.py
python -c "
path = r'C:\Users\<用户名>\AppData\Local\hermes\hermes-agent\gateway\status.py'
with open(path, 'r', encoding='utf-8') as f:
    content = f.read()
content = content.replace(
    'except (ProcessLookupError, PermissionError):',
    'except (ProcessLookupError, PermissionError, OSError):'
)
with open(path, 'w', encoding='utf-8') as f:
    f.write(content)
print('Done')
"
```

### 坑5：hermes 命令找不到

```powershell
# 手动添加PATH
$env:PATH += ";$env:USERPROFILE\.hermes\hermes-agent\venv\Scripts"
[Environment]::SetEnvironmentVariable("PATH", $env:PATH, "User")
# 重启终端
```

---

## 六、常用命令速查

| 命令 | 说明 |
|---|---|
| `hermes` | 启动对话 |
| `hermes --continue` | 恢复上次会话 |
| `hermes model` | 切换模型/供应商 |
| `hermes config` | 查看配置 |
| `hermes doctor` | 诊断问题 |
| `hermes gateway` | 启动飞书网关 |
| `hermes gateway setup` | 重新配置网关 |
| `hermes skills list` | 查看已装技能 |
| `hermes skills browse` | 浏览技能商店 |
| `hermes update` | 更新版本 |
| `/memory` | 对话中查看持久记忆 |
| `/skills` | 对话中查看技能 |
| `/model xxx` | 对话中切换模型 |

---

## 七、成本估算

| 项目 | 月费 |
|---|---|
| DeepSeek V4 Flash（日常） | ~30-50元 |
| DeepSeek V4 Pro（按需） | 按量计费 |
| 火山方舟 Coding Plan Lite（备选） | 40元/月 |
| 云服务器（7×24在线，可选） | ~100-200元/月 |
| **本地运行 Hermes** | **0元（自备电脑）** |

---

## 八、进阶：7×24 在线方案

本地电脑关机则飞书机器人断线。如需常在线：

- **云服务器部署**：华为云Flexus（~200元/月）、腾讯云Lighthouse（~100元/月）、Hostinger VPS
- **Docker 部署**：`docker run -it --rm -v ~/.hermes:/opt/data nousresearch/hermes-agent setup`
- 最低配置：2核4GB即可流畅运行

---

## 九、架构总览

```
飞书 App ←WebSocket→ Hermes Gateway ←API→ DeepSeek V4
                          ↓
                    持久记忆(SQLite)
                    技能系统(Skills)
                    终端执行(Terminal)
                    文件操作(File System)
```

---

## 十、阿里云云端部署方案（7×24在线）

> 推荐：轻量应用服务器一键部署，最低38元/年

### 方案一：轻量应用服务器一键部署（最省心 ⭐）

**购买地址**：https://www.aliyun.com/activity/ecs/clawdbot

1. 选择 Hermes Agent 官方镜像，2核4G起步
2. 地域选国内（华北2-北京 或 华东1-杭州）
3. 提交订单，系统自动部署
4. 部署完成后，控制台 → 实例 → 应用详情 → 配置 Hermes

**配置模型（DeepSeek V4）：**

```bash
# SSH 连接服务器后
sudo su root
hermes model
# 选 Custom Endpoint
# Base URL: https://api.deepseek.com/v1
# API Key: sk-xxxxxxxxxxxxxxxx
# Model: deepseek-chat
```

或编辑 `~/.hermes/.env`：

```env
DEEPSEEK_API_KEY=sk-xxxxxxxxxxxxxxxx
```

编辑 `~/.hermes/config.yaml`：

```yaml
model:
  provider: deepseek
  default: deepseek-chat
  deepseek:
    base_url: https://api.deepseek.com/v1
```

### 方案二：ECS云服务器手动部署

**适合**：已有ECS或需要更多自定义

```bash
# 1. SSH连接ECS（Ubuntu 22.04推荐）
ssh root@你的ECS公网IP

# 2. 更新系统
sudo apt update && sudo apt upgrade -y
sudo apt install -y ripgrep ffmpeg git

# 3. 一键安装Hermes（国内镜像）
curl -fsSL https://res1.hermesagent.org.cn/install.sh | bash

# 4. 刷新环境变量
source ~/.bashrc

# 5. 配置模型
hermes setup
# Quick setup → DeepSeek → 输入API Key

# 6. 验证
hermes --version
hermes doctor
```

### 方案三：计算巢弹性部署（按时计费）

**地址**：阿里云计算巢 → 搜索"HermesAgent社区版"

- 按需弹性，约0.254元/小时
- 适合临时体验，不用时可释放

### 方案四：无影云电脑（移动办公）

**地址**：阿里云无影云电脑 → 购买Hermes Agent一键部署

- 支持多端接入
- 需关闭"断连定时关机/休眠"才能7×24运行

### 阿里云端接入飞书

部署完成后，SSH连接服务器执行：

```bash
# 1. 配置飞书网关
hermes gateway setup
# 选 Feishu / Lark
# 填入 App ID、App Secret
# 来源：feishu
# 连接方式：websocket（阿里云有公网IP，也可以用webhook）

# 2. 设置环境变量
cat >> ~/.hermes/.env << 'EOF'
FEISHU_APP_ID=cli_xxxxxxxxxx
FEISHU_APP_SECRET=xxxxxxxxxxxxxxx
FEISHU_DOMAIN=feishu
FEISHU_CONNECTION_MODE=websocket
FEISHU_ALLOWED_USERS=ou_162d8a7a9558908aca3608a0c19f6ed7
FEISHU_GROUP_POLICY=open
EOF

# 3. 启动网关
hermes gateway

# 4. 设置开机自启（推荐用systemd）
sudo tee /etc/systemd/system/hermes-gateway.service << 'EOF'
[Unit]
Description=Hermes Agent Gateway
After=network.target

[Service]
Type=simple
User=root
WorkingDirectory=/root
ExecStart=/root/.local/bin/hermes gateway run
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable hermes-gateway
sudo systemctl start hermes-gateway

# 查看状态
sudo systemctl status hermes-gateway

# 查看日志
journalctl -u hermes-gateway -f
```

### 阿里云费用对比

| 方案 | 配置 | 月费 | 特点 |
|---|---|---|---|
| 轻量应用服务器 | 2核2G | 38元/年起 | 一键部署，最省心 |
| 轻量应用服务器 | 2核4G | 199元/年 | 推荐配置 |
| ECS按量 | 2核4G | ~100元/月 | 灵活 |
| 计算巢 | 弹性 | ~0.25元/小时 | 临时体验 |
| 无影云电脑 | 4核8G | ~100元/月 | 移动办公 |

---

## 十一、关键链接

| 资源 | 地址 |
|---|---|
| Hermes 中文安装镜像 | https://res1.hermesagent.org.cn/install.ps1 |
| Hermes 官方文档 | https://hermes-agent.nousresearch.com/docs/ |
| Hermes 飞书接入文档 | https://hermes-doc.aigc.green/user-guide/messaging/feishu |
| DeepSeek API | https://platform.deepseek.com |
| 飞书开放平台 | https://open.feishu.cn |
| 火山方舟 Coding Plan | https://www.volcengine.com/activity/codingplan |
| Hermes GitHub | https://github.com/NousResearch/hermes-agent |
