# Rule Evolution (规则进化与经验沉淀)

这是一个用于 AI 编程代理（Agents）的**元技能 (Meta-Skill)**，用于让 AI 在完成复杂的开发任务或修复大 Bug 后，**自动反思、提炼高价值的开发经验/网络防灾规范，并安全无冲突地合并进全局 `AGENTS.md` 规则文件中**，实现 Agent 的自我进化与心智沉淀。

---

## ✨ 核心特性

* **动态经验提炼**：将具体的代码修改或 Bug 排灾逻辑，自动抽象为长久生效的开发规范和避坑指南。
* **双向总结（正向规范 + 负向案例）**：除了写明“应该怎么做”，还会专门沉淀并记录“绝对不能做什么（Anti-Patterns）”及其历史断网/死锁的后果。
* **物理安全防御**：写入全局 `AGENTS.md` 前自动创建本地备份，并在写入后自动生成 `git commit` 进行版本锁定，防止改乱已有规则。
* **规则草案孵化**：新沉淀的零碎经验优先进入扩展锚点作为“草案规则（Draft Rules）”，经过后期验证后再合并进核心规约板块，防止规范库臃肿。

---

## 📦 支持的 AI 客户端与安装步骤

本技能遵循通用的 Agent 技能标准，支持以下主流 AI 客户端：

### 1. Google Antigravity / Gemini Extensions 用户
在你的终端中运行：
```bash
git clone https://github.com/你的用户名/rule-evolution.git ~/.gemini/config/skills/rule-evolution
```

### 2. Codex 客户端用户
在你的终端中运行：
```bash
git clone https://github.com/你的用户名/rule-evolution.git ~/.codex/skills/rule-evolution
```

### 3. Claude Code / Aider 等命令行 Agent 用户
本技能亦可以直接作为工程级开发准则直接合并，或者直接复制 [SKILL.md](SKILL.md) 放入你的项目 `.agents/` 目录中。

---

## ⚙️ 工作流程

当用户触发“使用 rule-evolution 整理经验”时，AI 代理会遵循以下步骤：
1. **提取经验**：分析当前会话日志（Transcript），梳理正面规范与负面案例。
2. **防冲突检测**：读取当前的全局 `AGENTS.md` 文件，去重并排除与已有主规约和草案条目的冲突。
3. **备份隔离**：在物理写入前，创建 `.bak` 备份并执行 Git 提交监控。
4. **合并写入**：将新提炼的经验追加写入到全局 `AGENTS.md` 底部的 `<!-- rule-evolution:draft-start/end -->` 草案规则定位锚点中。写入时需遵循**双语分层选择**（通用哲学架构用英文/中英专业对照，本地网络排灾配置用精炼中文）以及**「正反对比」立体格式**（正向规约 + Anti-Pattern 负面后果），验证无误后清理备份。
