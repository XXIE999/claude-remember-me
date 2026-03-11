# Remember Me

> 让你的 Claude Code 越用越懂你

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-Skill-blue)](https://claude.ai/code)

<p align="center">
  <img src="https://img.shields.io/badge/-%E4%B8%80%E5%8F%A5%E8%AF%9D%E4%B9%B0%E7%82%B9-orange" alt="一句话卖点"/>
</p>

**Remember Me** 是一个 Claude Code skill，能让 Claude 从"一次性助手"变成"越用越懂你的伙伴"。它会自动学习你的偏好、记住项目的技术决策、识别常用工作流，并在未来的对话中智能应用这些知识。

---

## 核心特性

### 零配置启动
安装后立即可用，自动从 `~/.claude/history.jsonl` 学习你的历史交互模式

### 渐进式学习
越用越智能，每次交互都可能学到新的偏好和工作流

### 跨项目复用
在一个项目中学到的知识（如 Python 打包流程）可以自动应用到类似项目

### 本地存储
所有记忆存储在本地 `~/.claude/remember-me.json`，隐私安全，数据不上传

### 导入导出
支持记忆的备份、恢复、分享 —— 把你的"AI 记忆"分享给朋友

---

## 快速开始

### 安装

将此 skill 添加到你的 Claude Code skills 目录：

```bash
# 方法 1: 直接复制到 skills 目录
cp -r remember-me ~/.claude/skills/

# 方法 2: 在 CLAUDE.md 中引用
echo '![remember-me](~/Desktop/skill/SKILL.md)' >> ~/.claude/CLAUDE.md
```

### 基础用法

```bash
# 学习最近的历史记录
/learn recent

# 手动添加一条记忆
/remember "这个项目使用 PyQt5，需要打包成 exe"

# 查询记忆
/recall "打包"

# 查看记忆库状态
/memory-status

# 导出记忆（备份或分享）
/memory-export

# 导入朋友的记忆模板
/memory-import my-friends-memory.json
```

---

## 命令一览

| 命令 | 功能 | 示例 |
|------|------|------|
| `/learn` | 从历史记录学习 | `/learn recent` |
| `/remember` | 手动添加记忆 | `/remember "偏好函数式风格"` |
| `/recall` | 检索记忆 | `/recall "deskpet"` |
| `/forget` | 删除记忆 | `/forget "过时的记忆"` |
| `/memory-status` | 查看记忆库状态 | `/memory-status` |
| `/memory-export` | 导出记忆 | `/memory-export backup.json` |
| `/memory-import` | 导入记忆 | `/memory-import backup.json` |

---

## 工作原理

### 智能学习流程

```
┌─────────────────────────────────────────────────────────────┐
│                    /learn 命令触发                          │
└─────────────────────┬───────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  Step 1: 读取 history.jsonl                                 │
│  - 按 timestamp 排序                                        │
│  - 按 project 分组                                          │
└─────────────────────┬───────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  Step 2: 识别项目边界                                       │
│  - project 字段变化 = 边界                                  │
│  - 同一 sessionId = 同一工作流                              │
└─────────────────────┬───────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  Step 3: 提取有价值内容                                     │
│  - 过滤低价值输入（问候、配置命令）                         │
│  - 识别技术决策、约束条件、偏好表达                         │
└─────────────────────┬───────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  Step 4: 构建工作流模板                                     │
│  - 识别迭代模式                                             │
│  - 提取可复用的工作流                                       │
└─────────────────────┬───────────────────────────────────────┘
                      ▼
┌─────────────────────────────────────────────────────────────┐
│  Step 5: 存储到 remember-me.json                           │
└─────────────────────────────────────────────────────────────┘
```

### 自动触发机制

**项目路径检测**：当检测到当前工作目录是已记忆的项目时，自动加载项目记忆。

**关键词触发**：当用户输入包含已学习的触发词时，自动应用相关模式。

```
用户: "帮我打包成exe"
🤖 检测到模式: python-packaging
  → 应用工作流: 检查依赖 → PyInstaller --noconsole → 测试 exe
```

---

## 记忆数据结构

```json
{
  "version": "1.0.0",
  "userProfile": {
    "preferredLanguage": "zh-CN",
    "codingStyle": "practical-first"
  },
  "projectMemories": {
    "deskpet": {
      "type": "python-desktop-app",
      "keyDecisions": ["使用 PyQt5", "PyInstaller 打包"],
      "constraints": ["不支持商城功能"]
    }
  },
  "learnedPatterns": [
    {
      "pattern": "python-packaging",
      "triggers": ["打包", "exe"],
      "recommendedAction": "使用 PyInstaller --noconsole"
    }
  ],
  "iterativeWorkflows": {
    "ui-adjustment": {
      "steps": ["考虑所有相关元素", "验证显示效果", "打包测试"]
    }
  }
}
```

---

## 使用场景

### 场景 1: 新项目快速上手

```bash
# 在旧项目中学到的知识
/remember "Python GUI 项目推荐 PyQt5，打包用 PyInstaller"

# 切换到新项目时
cd ~/Desktop/new-gui-project
# Claude 自动应用：推荐 PyQt5，提供打包方案
```

### 场景 2: 跨会话记忆

```bash
# 昨天
你: "角色设定：年，活泼，四川人，口头禅是'辣是种人生态度'"

# 今天（新会话）
你: "继续写年的对话"
# Claude 自动加载角色设定，保持一致
```

### 场景 3: 工作流学习

```bash
# 第一次打包（经历多次迭代）
你: "帮我打包" → 失败 → 修复 → 成功
# Remember Me 学习到完整的打包流程

# 下次打包
你: "帮我打包"
# Claude 自动应用学到的最佳实践，一次成功
```

### 场景 4: 分享记忆

```bash
# 导出你的记忆
/memory-export my-react-workflows.json

# 朋友导入
/memory-import my-react-workflows.json
# 朋友立即获得你积累的 React 开发经验
```

---

## 对比

| 特性 | 无 Remember Me | 有 Remember Me |
|------|----------------|----------------|
| 每次会话 | 从零开始 | 自动加载上下文 |
| 项目知识 | 每次重新说明 | 自动记住 |
| 工作流 | 每次重新学习 | 智能应用 |
| 偏好设置 | 每次重新表达 | 自动适应 |
| 经验分享 | 不可能 | 一键导出导入 |

---

## 文件结构

```
remember-me/
├── README.md                    # 本文档
├── SKILL.md                     # Skill 主文件
├── templates/
│   └── memory-structure.json    # 记忆存储结构模板
└── examples/
    └── example-memory.json      # 示例记忆文件
```

---

## 常见问题

**Q: 记忆会占用多少空间？**
A: 很少。典型的记忆文件只有几 KB，因为只存储关键信息，不存储完整对话。

**Q: 隐私安全吗？**
A: 完全安全。所有记忆存储在本地 `~/.claude/remember-me.json`，不上传任何服务器。

**Q: 记忆会过时吗？**
A: 可能会。建议定期使用 `/forget` 清理过时记忆，或使用 `/learn` 更新知识。

**Q: 可以选择性学习吗？**
A: 可以。使用 `/remember` 手动添加记忆，而不是 `/learn` 自动学习。

---

## 贡献

欢迎提交 Issue 和 Pull Request！

1. Fork 本仓库
2. 创建特性分支 (`git checkout -b feature/amazing-feature`)
3. 提交更改 (`git commit -m 'Add amazing feature'`)
4. 推送到分支 (`git push origin feature/amazing-feature`)
5. 创建 Pull Request

---

## 许可证

[MIT License](LICENSE)

---

<p align="center">
  <b>Remember Me - 让 Claude Code 记住你</b>
</p>

<p align="center">
  如果这个项目对你有帮助，请给一个 ⭐ Star！
</p>
