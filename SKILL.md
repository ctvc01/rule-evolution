---
name: rule-evolution
description: 分析当前对话的上下文和修改点，总结出有价值的经验、规则与行为约束，与现有的 AGENTS.md 进行比对去重后，按标准格式自动更新追加到规则文件中。
version: 1.2.0
author: Antigravity & Cheng
tags:
  - agent-rules
  - evolution
  - self-improvement
  - metadata-tracking
---

# Rule Evolution (规则进化与经验沉淀)

本技能旨在帮助 AI 代理在复杂、多轮的对话结束后，将当前会话中探索出的物理规律、排灾步骤、接口设计偏好等高价值经验，自动沉淀并固化到全局开发规范文件 [AGENTS.md](file:///Users/cheng/.gemini/config/AGENTS.md) 中，实现 Agent 规则的“自我进化”。

## 🎯 适用场景 (When to Use)

- 当用户明确发出指令，要求“将刚才解决的问题总结并更新到规则中”时。
- **主动触发启发式 (Trigger Heuristics)**：当本次会话非用户手动发起总结、但满足以下量化指标之一时，AI 代理**必须在会话结束前主动询问用户**是否需要沉淀规则（“主动提醒，但把关权保留给用户”，与全自动黑盒写入区分开）：
  - **曲折调试链路**：会话中发生了 3 次及以上的报错/失败与反复调试（如 `Error -> Debug -> Retry` 循环）。
  - **多文件重构**：本次开发中，修改了 3 个及以上的核心代码文件。
  - **关键架构变更**：确立了新的 API 接口规范、引入了新的核心依赖，或改变了系统既有的部署/安全架构。

## ⚙️ 核心工作流 (Core Workflow)

AI 代理在加载本技能后，必须严格按照以下五个步骤执行：

### 第一步：Review 历史与提取经验 (Extract Experience)
1. **分析上下文**：使用 `view_file` 或相关工具查看当前会话的完整历史或 `transcript.jsonl` 日志，梳理解决过的问题、做过的关键代码修改（Diff）以及配置调整。
2. **双向经验提炼**：将这些具体的改动提炼为**通用的、面向未来开发的**规范：
   - **正面指导原则**：应该怎么做，如何规范设计。
   - **血泪负面案例 (Anti-Patterns)**：绝对不能怎么做，记录当时断网/死锁的具体物理表现和导致原因。
   * *负面案例范例*：“在 Merge.yaml 中排除了 10.58.131.125，但漏掉了 10.0.0.0/8。后果：导致 Tun 模式下其余内网域名走代理，被公司 WAF 直接拦截而断网。”

### 第二步：对比去重与防冲突 (Conflict & Duplicate Check)
1. **跨会话关键词预检索 (Pre-Retrieval via grep)**：
   - 从待提炼的经验中提取 2-3 个核心实体词（如 `Clash`, `DNS`, `Timeout`, `Tunnel`）。
   - 在终端使用 grep 命令行工具对全局规则文件 [AGENTS.md](file:///Users/cheng/.gemini/config/AGENTS.md)（以及历史备份或归档文件 `AGENTS.archive.md`，如果存在的话）进行关键词快速匹配，检测是否已有类似主题：
     ```bash
     grep -i -E "(Clash|DNS|Timeout)" /Users/cheng/.gemini/config/AGENTS.md
     ```
   - 收集匹配到的历史行，用于接下来的去重分析。
2. **读取现有的 `AGENTS.md`**：读取系统当前的 [AGENTS.md](file:///Users/cheng/.gemini/config/AGENTS.md)。
3. **比对与防噪**：
   - **去重**：结合 grep 检索到的关联条目，全面比对已有的主分类规约以及草案/偏好区域。若新规则与现有任意条目有等价、包含或高度重叠的表述，则直接跳过，防止规则库无序膨胀。
   - **防冲突**：若提炼的新规则与已有的规则在逻辑上冲突，严禁直接覆盖或强行插入。必须列出冲突项，向用户阐明冲突原因，并在获得授权后才能修改或弃用旧规则。

### 第三步：自动备份与版本回滚防护 (Pre-Write Protection)
1. **本地备份**：在物理写入 [AGENTS.md](file:///Users/cheng/.gemini/config/AGENTS.md) 之前，必须在同目录下创建一个名为 `AGENTS.md.bak` 的临时备份，以防写入时发生异常或进程崩溃损坏原文件。
2. **Git 版本标记**：完成修改且验证无误后，如果该目录处于 Git 仓库中，在终端执行 Git 提交，对规则更新进行版本锁定：
   ```bash
   git add <agents.md_path> && git commit -m "chore(agents): 自动规则沉淀 - [简短规则主题描述]"
   ```

### 第四步：按分类与三层锚点归类写入 (Formatting & Multi-Layer Anchor)
1. **精确定位与锚点选择**：在写入 [AGENTS.md](file:///Users/cheng/.gemini/config/AGENTS.md) 时，根据规则的性质精确定位至以下三层锚点对之一：
   - **核心规约层**：`<!-- rule-evolution:core-start -->` / `<!-- rule-evolution:core-end -->`（通常用于人工验证成熟后的晋升写入，或关键性的全局通用硬规约）。
   - **草案孵化层**：`<!-- rule-evolution:draft-start -->` / `<!-- rule-evolution:draft-end -->`（默认写入层，用于沉淀本次会话产生的临时、局部项目经验）。
   - **个人偏好层**：`<!-- rule-evolution:user-prefs-start -->` / `<!-- rule-evolution:user-prefs-end -->`（用于存放开发者个人的编码习惯、命名偏好或工具偏好，与其他客观工程规范隔离）。
   新追加的条目**必须**写入对应注释锚点对的内部。
2. **双语分层选择 (Language Selection)**：
   - **通用开发哲学与代码架构规则**（面向逻辑思维与代码生成）：优先使用**英文**或**中英对照的专业术语**（例如 `YAGNI`, `Root Cause`），以对齐 LLM 预训练语料，达到最高理解精度。
   - **本地环境、网络运维、配置安全与物理操作规则**（面向排灾与特定工具链）：统一使用**规范、精炼的中文**，保留具体参数和命令，防止翻译概念偏差，并便于人类审查。
3. **元数据识别与溯源 (Traceability Metadata)**：写入新规则时，在规则行尾强制追加带有标准追踪元数据的 HTML 注释（hits 初始值为 1，日期使用 UTC+8 时区）：
   格式：`<!-- hits:1 created:YYYY-MM-DD session:xxxxxx -->`
4. **双向立体格式**：每一条新加入的规则必须遵循“正反对比”的双向立体句式，使规则具备高实操防呆性：
   `* **[核心关键词/Core Keyword]**：[正向执行规约/正确做法]。反面避坑 (Anti-Pattern)：[负向案例表现、导致原因及物理后果] <!-- hits:1 created:YYYY-MM-DD session:xxxxxx -->`

### 第五步：安全更新与持久化验证 (Safe Update & Verify)
1. **写入与清理**：在锚点间追加成功后，使用 `cat` 或 `grep` 进行验证，确保没有破坏 HTML 锚点注释。验证无误后，删除临时备份 `AGENTS.md.bak` 文件。

### 第六步：草稿晋升评估与自动优化 (Promotion Check & Optimization)
1. **Hits 命中统计**：在每次会话启动、AI 读取 `AGENTS.md` 时，如果 AI 判定某条 Draft（草稿层）或 User Prefs（偏好层）规则对本次代码改动有直接的指导或避免了报错，则在沉淀更新时，顺便将该规则行尾的 `hits:N` 计数器自增 1。
2. **自动晋升机制**：当某条草稿层规则的 `hits` 达到 3 次时，AI 代理应当在会话结束时，主动输出 Nudge 提示用户：“检测到草稿规则『XXX』近期已被多次有效命中，建议将其晋升至核心规约层（Core Rules）或分类主板块。”并在用户同意后，在下次沉淀时自动将其移动至 `core` 锚点或核心主分类中，清空原草稿位置。
