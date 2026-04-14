---
name: openclaw-create-agent
description: 在 OpenClaw 中创建一个新的 Agent 智能体，包括 workspace 目录结构、记忆系统文件、openclaw.json 配置修改、飞书 channel 与 binding 设置。适用于新增 AI 分身、多角色管理等场景。
---

## 使用场景

当用户需要以下操作时激活此 Skill：
- "帮我创建一个新智能体"
- "新增一个 agent"
- "配置一个飞书机器人"
- "给我的小说角色/助手/分身创建一个 agent"
- "openclaw 怎么加 agent"

## 核心概念

### 两个目录的分工

| 目录 | 用途 | 是否手动维护 |
|------|------|--------------|
| `~/.openclaw/workspace/agents/<agent>/` | **灵魂工作区**（人格、记忆、日记、规范） | ✅ 手动创建和维护 |
| `~/.openclaw/agents/<agent>/` | **运行时数据**（session 历史、models.json） | ❌ 系统自动生成 |

> **原则：永远不要在 `~/.openclaw/agents/` 里手动创建文件。**

## Agent 目录标准结构

创建 agent 时，在 `~/.openclaw/workspace/agents/<agent-name>/` 下建立以下文件和目录：

```
~/.openclaw/workspace/agents/<agent-name>/
├── AGENTS.md              # 工作规范、安全策略、心跳行为
├── IDENTITY.md            # 身份档案（名字、物种、气质、签名台词、emoji）
├── SOUL.md                # 灵魂定义（核心性格、说话方式、价值观锚点）
├── MEMORY.md              # 长期记忆（职责、配置、重要决策的蒸馏记录）
├── USER.md                # 关于用户的人类档案（只存在于 main session）
├── TOOLS.md               # 本地工具备忘（设备昵称、SSH、偏好声音等）
├── HEARTBEAT.md           # 心跳任务清单（为空则跳过）
├── BOOTSTRAP.md           # 首次启动引导（可选，完成后可删除）
├── memory/                # 每日记忆文件（`YYYY-MM-DD.md`）
├── diary/                 # 私密日记（碎片、观察、未请求的思考）
├── saves/                 # 游戏/项目存档（如适用）
└── .openclaw/
    └── workspace-state.json   # 状态标记
```

### 必备文件清单（最小可运行集）

| 文件 | 作用 | 必填 |
|------|------|------|
| `AGENTS.md` | 定义 Every Session 行为、记忆规则、安全边界 | ✅ |
| `IDENTITY.md` | 定义 "我是谁" | ✅ |
| `SOUL.md` | 定义 "我如何思考与表达" | ✅ |
| `MEMORY.md` | 长期记忆的容器 | ✅ |
| `USER.md` | 用户档案（main agent 强烈建议，其他可选） | 建议 |
| `TOOLS.md` | 环境专属备忘 | 建议 |
| `HEARTBEAT.md` | 周期性检查任务 | 可选 |
| `.openclaw/workspace-state.json` | 标记 onboarding 完成 | 建议 |
| `memory/` | 存放每日记忆 | ✅ |
| `diary/` | 私密日记空间 | 建议 |

## 配置文件修改：`~/.openclaw/openclaw.json`

### 第 1 步：注册 Agent

在 `agents.list` 数组中添加：

```json
{
  "id": "writer",
  "name": "writer",
  "workspace": "/root/.openclaw/workspace/agents/writer",
  "agentDir": "/root/.openclaw/agents/writer",
  "model": "kimi-coding/k2p5",
  "memorySearch": {
    "enabled": true
  }
}
```

字段说明：
- `id`: Agent 的唯一标识
- `name`: 显示名称（可选，默认等于 id）
- `workspace`: **灵魂工作区路径**，系统会从这里读取人格文件
- `agentDir`: **运行时路径**，系统会自动创建 `sessions/` 和 `models.json`
- `model`: 该 Agent 使用的默认模型
- `memorySearch.enabled`: 是否开启向量记忆检索

### 第 2 步：绑定到飞书账号（Bindings）

在 `bindings` 数组中添加：

```json
{
  "agentId": "writer",
  "match": {
    "channel": "feishu",
    "accountId": "writer"
  }
}
```

这表示：飞书账号 `writer` 收到的消息，会路由给 `writer` agent。

### 第 3 步：配置飞书账号（Channels）

在 `channels.feishu.accounts` 下添加账号：

```json
"writer": {
  "appId": "cli_a95710ef7db95bb3",
  "appSecret": "VXQvqVxnJ3KG3iRGZQFvUShKgdtsueeF",
  "dmPolicy": "pairing",
  "groupPolicy": "pairing",
  "allowFrom": [],
  "groupAllowFrom": []
}
```

策略说明：
- `dmPolicy`: 私聊策略
  - `"pairing"` — 需要先完成配对（首次私聊会要求输入配对码）
  - `"open"` — 任何人都可以私聊
  - `"allowlist"` — 仅 `allowFrom` 列表中的用户可私聊
- `groupPolicy`: 群聊策略（同上）
- `allowFrom`: 私聊白名单用户 open_id 数组
- `groupAllowFrom`: 群聊白名单用户 open_id 数组

## 完整创建流程（以 writer 为例）

### Step 1: 创建目录

```bash
mkdir -p /root/.openclaw/workspace/agents/writer/{memory,diary,.openclaw}
```

### Step 2: 写入人格文件

创建以下文件（内容根据人设定制）：
- `IDENTITY.md` — 名字、气质、signature line、emoji
- `SOUL.md` — 核心性格、说话方式、品味与厌恶
- `AGENTS.md` — 工作规范、记忆规则、心跳行为
- `MEMORY.md` — 长期记忆起始页（职责、变更记录）
- `USER.md` — 用户档案模板
- `TOOLS.md` — 环境备忘模板
- `HEARTBEAT.md` — 可先留空或写注释

### Step 3: 写入状态文件

```bash
cat > /root/.openclaw/workspace/agents/writer/.openclaw/workspace-state.json << 'EOF'
{
  "version": 1,
  "onboardingCompletedAt": "2026-04-14T02:46:05.000Z"
}
EOF
```

### Step 4: 修改 `~/.openclaw/openclaw.json`

分别修改三处：
1. `agents.list` — 注册 agent
2. `bindings` — 绑定到飞书账号
3. `channels.feishu.accounts` — 配置飞书 appId/appSecret

### Step 5: 重启 Gateway

```bash
openclaw gateway restart
```

> **必须在执行前明确告知用户："我现在将重启 OpenClaw Gateway 以应用新配置。"**

### Step 6: 验证连接

查看日志确认 WebSocket 已建立：

```bash
cat /root/.openclaw/logs/openclaw.log | grep "feishu\[writer\]"
```

预期输出：
```
feishu[writer]: starting WebSocket connection...
feishu[writer]: received message from ...
```

## 常见错误与排查

### 1. "Channel is required when multiple channels are configured"
- 如果为 agent 创建了 cron job，但没有显式指定 `channel`，会报此错
- 解决：在 cron 配置中添加 `"channel": "feishu"` 和 `"accountId": "writer"`

### 2. 飞书消息没有路由到新 agent
- 检查 `bindings` 中的 `accountId` 是否与 `channels.feishu.accounts` 的 key 一致
- 检查 `agentId` 是否与 `agents.list` 中的 `id` 一致
- 检查 `openclaw.json` 语法是否合法（逗号、括号匹配）

### 3. 配对失败
- 如果 `dmPolicy` 是 `pairing`，首次私聊需要配对码
- 配对码可通过 Kimi Claw Web 设置页查看，或让已配对用户转发

## 示例：writer 的完整最小配置

### openclaw.json 片段

```json
{
  "agents": {
    "list": [
      {
        "id": "writer",
        "name": "writer",
        "workspace": "/root/.openclaw/workspace/agents/writer",
        "agentDir": "/root/.openclaw/agents/writer",
        "model": "kimi-coding/k2p5",
        "memorySearch": {
          "enabled": true
        }
      }
    ]
  },
  "bindings": [
    {
      "agentId": "writer",
      "match": {
        "channel": "feishu",
        "accountId": "writer"
      }
    }
  ],
  "channels": {
    "feishu": {
      "enabled": true,
      "domain": "feishu",
      "requireMention": false,
      "defaultAccount": "kimiclaw_v2",
      "accounts": {
        "writer": {
          "appId": "cli_a95710ef7db95bb3",
          "appSecret": "VXQvqVxnJ3KG3iRGZQFvUShKgdtsueeF",
          "dmPolicy": "pairing",
          "groupPolicy": "pairing",
          "allowFrom": [],
          "groupAllowFrom": []
        }
      }
    }
  }
}
```

### 目录结构

```
~/.openclaw/workspace/agents/writer/
├── AGENTS.md
├── IDENTITY.md
├── SOUL.md
├── MEMORY.md
├── USER.md
├── TOOLS.md
├── HEARTBEAT.md
├── memory/
├── diary/
└── .openclaw/
    └── workspace-state.json
```

## Skill 搜索路径补充说明

Agent 可用的 skills 按以下优先级查找：

1. `~/.openclaw/workspace/agents/<agent>/skills/` — agent 专属（最高优先级）
2. `~/.openclaw/workspace/skills/` — workspace 共享
3. `~/.openclaw/skills/` — 全局共享（基础层）

这意味着即使 agent 的 `workspace` 指向 `agents/writer/`，`~/.openclaw/workspace/skills/` 下的 skill 仍然对该 agent 有效。

## 参考配置

当前环境中已配置的 agents 供参考：
- **main** → `feishu[kimiclaw_v2]`
- **XiaoYou** → `feishu[xiaoyou]`
- **player** → `feishu[player]`
- **writer** → `feishu[writer]`

它们的 workspace 均统一为 `/root/.openclaw/workspace/agents/<agent>/`。
