# Remember Me - Claude Code 持久记忆 Skill

## 简介

Remember Me 是一个让 Claude Code 从"一次性助手"变成"越用越懂你的伙伴"的 skill。它能：

- 从历史交互记录中自动学习用户偏好和项目知识
- 跨会话持久化存储记忆
- 智能识别项目上下文并自动应用相关知识
- 支持记忆的导入导出，方便备份和分享

## 记忆存储位置

记忆文件存储在：`~/.claude/remember-me.json`

## 命令列表

### `/learn [scope]` - 学习历史记录

分析 `~/.claude/history.jsonl` 提取知识并存储到记忆库。

**参数**：
- `recent` - 只分析最近 50 条记录（默认）
- `full` - 分析全部历史记录
- `<project_name>` - 只分析特定项目

**示例**：
```
/learn recent
/learn full
/learn deskpet
```

**学习流程**：
1. 读取并按时间排序历史记录
2. 按 project 字段分组识别项目边界
3. 过滤低价值输入（问候、配置命令）
4. 提取技术决策、约束条件、偏好表达
5. 识别迭代式工作流模式
6. 存储到 remember-me.json

---

### `/remember <内容>` - 手动添加记忆

手动添加一条记忆到记忆库。

**示例**：
```
/remember "这个项目使用 PyQt5，需要打包成 exe"
/remember "用户偏好简洁的代码风格"
/remember "角色设定：年，活泼，四川人"
```

**记忆类型自动识别**：
- 包含项目路径 → 项目记忆
- 包含"偏好"、"喜欢" → 用户偏好
- 包含技术名词 → 技术决策

---

### `/recall <关键词>` - 检索记忆

根据关键词检索已存储的记忆。

**示例**：
```
/recall deskpet
/recall 打包
/recall 角色设定
```

**输出格式**：
```
📂 项目记忆: deskpet
  - 类型: python-desktop-app
  - 技术栈: PyQt5
  - 约束: 不支持商城、游戏功能

🔄 学习模式: python-packaging
  - 触发词: 打包, exe, PyInstaller
  - 推荐行为: 使用 PyInstaller 打包，隐藏控制台窗口

📋 偏好规则: chinese-first
  - 条件: communication
  - 行为: 优先使用中文回复
```

---

### `/forget <关键词或ID>` - 删除记忆

删除指定记忆。可以按关键词匹配或精确 ID 删除。

**示例**：
```
/forget "deskpet"
/forget "id:project-deskpet-001"
```

**安全确认**：
- 删除前会显示匹配的记忆
- 需要用户确认后才执行删除

---

### `/memory-status` - 查看记忆库状态

显示当前记忆库的整体状态。

**输出示例**：
```
🧠 Remember Me - 记忆库状态
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

📊 统计信息:
  - 记忆版本: 1.0.0
  - 最后更新: 2026-03-12
  - 项目记忆: 3 个
  - 学习模式: 5 个
  - 偏好规则: 2 个
  - 工作流模板: 4 个

📂 已记忆项目:
  1. deskpet (Python 桌面应用)
  2. cc (文档处理)
  3. my-website (前端项目)

🔄 最近学习的模式:
  - python-packaging
  - ui-adjustment
  - novel-writing
```

---

### `/memory-export [文件名]` - 导出记忆

导出记忆到 JSON 文件，用于备份或分享。

**参数**：
- 不指定文件名 → 导出到 `~/.claude/remember-me-export-YYYYMMDD.json`
- 指定文件名 → 导出到指定路径

**示例**：
```
/memory-export
/memory-export my-memories.json
/memory-export ~/Downloads/memories.json
```

---

### `/memory-import <文件路径>` - 导入记忆

从 JSON 文件导入记忆，与现有记忆合并。

**合并策略**：
- 相同 key 的记忆 → 保留较新的版本
- 新 key 的记忆 → 直接添加
- 冲突时 → 询问用户选择

**示例**：
```
/memory-import remember-me-export-20260312.json
/memory-import ~/Downloads/friend-memories.json
```

---

## 自动触发机制

### 项目路径检测

当检测到当前工作目录是已记忆的项目时，自动加载项目记忆：

```
检测到项目: deskpet
📚 已加载项目记忆:
  - 技术栈: PyQt5
  - 约束: 不支持商城、游戏功能
  - 角色设定: 年，活泼，四川人
```

### 关键词触发

当用户输入包含已学习的触发词时，自动应用相关模式：

```
用户: "帮我打包成exe"
🤖 检测到模式: python-packaging
  → 应用工作流: 检查依赖 → PyInstaller --noconsole → 测试 exe
```

### 上下文延续

当用户说"继续"、"上次我们..."时，自动加载相关上下文：

```
用户: "继续上次的工作"
📚 加载最近会话上下文:
  - 项目: deskpet
  - 上次任务: UI 布局调整
  - 进度: 已完成间距调整，待打包测试
```

---

## 记忆数据结构

记忆文件 `~/.claude/remember-me.json` 的结构：

```json
{
  "version": "1.0.0",
  "lastUpdated": "2026-03-12T00:00:00Z",
  "userProfile": {
    "preferredLanguage": "zh-CN",
    "codingStyle": "practical-first",
    "communicationStyle": "direct-concise"
  },
  "projectMemories": {},
  "learnedPatterns": [],
  "preferenceRules": [],
  "iterativeWorkflows": {}
}
```

详细结构见 `templates/memory-structure.json`。

---

## 实现指令

当用户调用此 skill 时，Claude 应该：

1. **解析命令**：识别用户想要执行的操作
2. **执行操作**：根据命令类型执行相应逻辑
3. **反馈结果**：以友好的格式展示操作结果
4. **更新记忆**：如有需要，更新 `~/.claude/remember-me.json`

### 核心实现逻辑

#### 学习历史记录 (`/learn`)

```
1. 读取 ~/.claude/history.jsonl
2. 按参数过滤记录（recent/full/project）
3. 按 project 字段分组
4. 对每组记录：
   a. 识别项目边界
   b. 提取有价值内容（过滤问候、配置命令）
   c. 识别技术决策、约束条件
   d. 构建工作流模板
   e. 识别迭代模式
5. 合并到现有记忆文件
6. 输出学习结果摘要
```

#### 有价值内容识别

高价值特征（保留）：
- 技术决策："使用 PyQt5"、"选择 React"
- 约束条件："需要打包成 exe"、"不支持商城"
- 偏好表达："我想要简洁的代码"、"偏好函数式风格"
- 角色设定：粘贴的角色背景、性格描述
- 工作流模式："帮我打包"、"检查一下"

低价值特征（过滤）：
- 简单问候："你好"、"hi"
- 配置命令："/config"、"/plugin"
- 临时调试：代理设置、环境变量
- 无关对话：随意聊天

#### 迭代式工作流识别

```
1. 检测迭代信号词：
   - "检查"、"修复"、"修改"、"不对"、"不行"
   - "重写"、"继续"、"再试"、"换一种"

2. 追踪同一 sessionId 内的连续修改

3. 提取迭代模式：
   - 每次迭代的变化点
   - 用户不满意的原因
   - 最终成功的方案

4. 生成工作流模板
```

---

## 注意事项

1. **隐私保护**：所有记忆存储在本地，不上传到任何服务器
2. **记忆容量**：定期清理过时记忆，保持记忆库精简
3. **冲突处理**：导入记忆时注意处理冲突
4. **备份建议**：定期使用 `/memory-export` 备份记忆

---

## 版本历史

- v1.0.0 (2026-03-12): 初始版本
  - 基础记忆存储和检索
  - 历史记录学习
  - 项目记忆自动加载
  - 记忆导入导出
