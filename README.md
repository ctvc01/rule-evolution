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

## ⚙️ 工作流程 (Plan-then-Execute 两阶段确认)

本技能的运行严格划分为**规划阶段**（只读提案）与**执行阶段**（可写确认），确保对全局规约文件的物理修改处于绝对控制下。

---

### 阶段一：规划阶段 (Phase 1: Plan) — 提炼与审查 (只读)
*此阶段 Agent 决不执行任何物理文件写入或 Git 提交。*

0. **主动触发 (Trigger Heuristics)**：当本次开发经历曲折调试（报错 $\ge 3$ 次）、多文件修改（$\ge 3$ 个核心文件）或重大架构变更时，Agent 主动 nudge 提示用户沉淀规则。
1. **提取与去背景抽象 (Extract & Abstract)**：
   - 分析会话，剥离所有特定项目的具象细节（IP、MAC、自研函数名、厂牌特有名等），升级提炼为高阶通用的技术模式规范。
   - 采用正面标准执行指引与 Anti-Pattern 负面后果的“正反对比”立体句式描述。
2. **查重与过滤被拒历史 (Conflict & Rejection Check)**：
   - 提取实体词执行 `grep` 检索。
   - 对比主分类、草稿区、偏好区以及**拒绝记录区 (Rejected Rules)**，凡是已存在或曾被用户拒绝（或测试失败）的废弃经验，直接过滤跳过。
3. **提案展现与强制暂停 (Proposal & Nudge)**：
   - 展现规则草案及拟写入分区（Core / Draft / User Prefs），标明与已有条目的差异。
   - 输出 Proceed 呼吁并**强制结束 Turn，停止所有工具调用**，等待用户指示。

---

### 阶段二：执行阶段 (Phase 2: Execute) — 测试与物理写入 (可写)
*仅在用户针对规划提案明确输入 “Proceed”、“执行” 或 “确认” 之后，Agent 才会恢复运行并被授权调用修改写入类的工具。*

4. **物理备份与隔离 (Pre-Write Protection)**：物理写入前自动创建 `AGENTS.md.bak` 备份并锁定 Git 提交。
5. **本地验证门与分层写入 (Dry-Run Gate & Write)**：
   - **验证门检测**：物理写入前，自动在终端执行项目非交互测试套件（如 `npm run test`, `pytest`）。若测试失败，认为新规则具有破坏性，**拦截写入**，发出警报并自动将其归档到拒绝记录区。
   - **分层写入**：测试通过后，精准追加到目标锚点，行尾追加追踪元数据标签（hits/created/session）。清除备份。
   - **Git 锁定**：自动执行 Git commit 提交锁定版本。
6. **晋升、归档与打包分裂 (Promotion & Graduation)**：
   - **拒绝归档**：若用户在阶段一拒绝 Proposal 提案，Agent 在第二阶段自动将其写入拒绝记录区以备忘过滤。
   - **自动晋升**：草稿命中 `hits` 满 3 次时，nudge 提示并协助用户将其剪切晋升至核心主板块。
   - **细胞分裂 (Graduation)**：当草稿区或偏好区中，针对同一垂直领域的条目积累达 3 条及以上且 Hits 较高时，提示并自动将其打包分裂为独立的 Skill 文件夹（生成符合 `agentskills.io` 规范的 `SKILL.md`），使全局规则库保持极简。
