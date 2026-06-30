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

在日常开发及会话结束前，AI 代理会遵循以下步骤实现规则进化：

0. **主动触发 (Trigger Heuristics)**：当本次开发经历曲折调试（报错 $\ge 3$ 次）、多文件修改（$\ge 3$ 个核心文件）或发生重大架构变更时，Agent 会主动 nudge 提示用户进行经验沉淀。
1. **提取经验 (Extract)**：分析会话历史，梳理出面向未来的正面规范与防呆负面案例。
2. **去重与查重 (Conflict & Duplicate Check)**：
   - 提取规则实体词，使用终端 `grep` 在全局规则中进行跨会话关联检索；
   - 对比主分类、草案区、偏好区以及**拒绝记录区 (Rejected Rules)**，凡是已存在或曾被用户拒绝（或测试失败）的废弃经验，直接过滤跳过。
3. **备份隔离 (Pre-Write Protection)**：物理写入前自动创建 `AGENTS.md.bak` 备份并锁定 Git 提交。
4. **合并与三层锚点写入 (Formatting & Multi-Layer Anchor)**：
   - 依据规则属性精准写入 `core`、`draft` 或 `user-prefs` 对称锚点区间；
   - 遵循**双语分层**（哲学/架构英文，本地环境/网络中文）与**「正反对比」立体格式**写入；
   - 行尾强制追加元数据标记：`<!-- hits:1 created:YYYY-MM-DD session:xxxxxx -->`。
5. **安全校验与本地验证门 (Safe Update & Dry-Run Gate)**：
   - 物理写入前，自动运行本地测试命令（如 `npm run test`, `pytest` 等）。若测试失败（Exit Code $\ne 0$），则认为新规则具有破坏性，拦截写入并自动归档到拒绝记录区。
   - 测试通过后，追加新规则并清理备份。
6. **晋升、归档与打包分裂 (Promotion & Graduation)**：
   - **拒绝归档**：若用户拒绝沉淀某规则，自动将其归入拒绝记录区以备忘过滤。
   - **自动晋升**：草稿命中计值 `hits` 达 3 次时，Agent nudge 提示并协助用户晋升至核心主板块。
   - **细胞分裂**：当草稿区或偏好区中，针对同一垂直领域的条目积累达 3 条及以上且 Hits 较高时，AI 会提示并自动将其打包分裂为独立的 Skill 文件夹（生成符合 `agentskills.io` 规范的 `SKILL.md`），使规则库保持精简。
