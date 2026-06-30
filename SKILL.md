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

## ⚙️ 核心工作流 (Core Workflow - Plan-then-Execute)

AI 代理加载本技能后，**必须且只能**将工作流严格划分为以下两个独立会话阶段执行。决不允许越过规划阶段直接写入规则文件。

---

### 阶段一：规划阶段 (Phase 1: Plan) — 提炼与用户核对 (只读)
*在规划阶段，AI 代理只允许使用 `view_file`、`grep` 等只读工具对全局规则进行检索和查重，**绝对禁止**运行任何物理写入工具 (如 `replace_file_content`, `write_to_file`) 或 Git 提交命令。*

#### 第一步：Review 历史与提取经验 (Extract Experience)
1. **分析上下文**：使用 `view_file` 或相关工具查看当前会话的完整历史或 `transcript.jsonl` 日志，梳理解决过的问题、做过的关键代码修改（Diff）以及配置调整。
2. **去背景抽象化 (Context-Free Abstraction)**：
   在提炼规则时，必须清除所有特定项目的具象细节，进行高阶技术模式抽象，使其能零成本复用到任何其他同类技术栈项目中。
   - **禁止**：包含特定 IP 地址/网段（如 `10.58.131.125`）、特定的物理 MAC 地址、特定的自研函数/组件名（如 `updateClashRules`）、特定的局部厂牌特有名称（如阿里云/腾讯云 CDN）。
   - **推荐**：将其抽象为通用名词（如：`局域网专用 CIDR 范围`、`客户端物理并发标识`、`耗时异步硬件重载接口`、`国内头部公共 CDN 节点 IP`）。
3. **双向立体经验提炼**：将这些抽象后的变动提炼为具有普适指导意义的立体规范：
   - **正面指导原则**：通用设计模式下的正确执行做法。
   - **血泪反面案例 (Anti-Patterns)**：通用场景下的错误操作、发生原因及导致的系统级物理后果（如死锁、连接暴跌、长时间服务阻断等）。
   - **抽象对照范例**：
     * *具象（不推荐）*：`对 enableAcceleration 接口在路由层用 Map 缓存去重，防同 MAC 连续调用导致 Clash 重启网络中断`
     * *抽象（推荐）*：`耗时异步硬件重载写操作必须在路由接收层基于客户端标识进行并发去重与 Promise 缓存，合并重复的并发请求。反面避坑 (Anti-Pattern)：若未去重，高频并发重载会导致底层物理守护进程被连续多次杀灭重启，引发严重的端口竞争、死锁与服务长时间阻断。`

#### 第二步：对比去重与防冲突 (Conflict & Duplicate Check)
1. **跨会话关键词预检索 (Pre-Retrieval via grep)**：
   - 从待提炼的经验中提取 2-3 个核心实体词（如 `Clash`, `DNS`, `Timeout`, `Tunnel`）。
   - 在终端使用 grep 命令行工具对全局规则文件 [AGENTS.md](file:///Users/cheng/.gemini/config/AGENTS.md)（以及历史备份或归档文件 `AGENTS.archive.md`，如果存在的话）进行关键词快速匹配，检测是否已有类似主题：
     ```bash
     grep -i -E "(Clash|DNS|Timeout)" /Users/cheng/.gemini/config/AGENTS.md
     ```
   - 收集匹配到的历史行，用于接下来的去重分析。
2. **读取现有的 `AGENTS.md`**：读取系统当前的 [AGENTS.md](file:///Users/cheng/.gemini/config/AGENTS.md)。
3. **比对与防噪**：
   - **去重与过滤被拒历史 (Deduplication & Rejection Filtering)**：结合 grep 检索到的关联条目，全面比对已有的主分类规约、草案区、偏好区以及**拒绝记录区 (Rejected Rules)**。如果待写入的新规则与拒绝记录区中的记录内容高度相似（代表用户曾拒绝过或因测试失败被抛弃），或与现有任意条目有等价、包含或重叠的表述，则直接跳过，防止规则库无序膨胀和重复提议。
   - **防冲突**：若提炼的新规则与已有的规则在逻辑上冲突，严禁直接覆盖或强行插入。必须列出冲突项，向用户阐明冲突原因，并在获得授权后才能修改或弃用旧规则。

#### 第三步：提案展现与拦截等待 (Proposal & Nudge)
1. **生成规则草案**：将提炼好的规范（格式需符合“双语分层选择”和“双向立体格式”）以及拟写入的目标分区（Core / Draft / User Prefs）整理为一份详细的 Proposal。
2. **防冗余展现**：明确列出 `grep` 检索对比结果，指出相似的历史条目，并阐述本次写入的补充意义。
3. **强制暂停（终结 Turn）**：在提案最后输出标准的 Proceed 呼吁：
   `【已完成规则提炼与规划，正在等待您的 Proceed 指令以执行测试验证与物理写入。在此之前我将保持只读，决不修改任何文件】`
   AI 代理**必须在此处结束当前 Turn，停止所有工具调用**，等待用户回复。

---

### 阶段二：执行阶段 (Phase 2: Execute) — 测试与物理写入 (可写)
*仅在用户针对上述提案明确输入 “Proceed”、“执行” 或 “确认” 之后，AI 代理才可恢复运行并执行以下修改与写入工具。*

#### 第四步：自动备份与版本防护 (Pre-Write Protection)
1. **本地备份**：在物理写入 [AGENTS.md](file:///Users/cheng/.gemini/config/AGENTS.md) 之前，必须在同目录下创建一个名为 `AGENTS.md.bak` 的临时备份，以防写入时发生异常或进程崩溃损坏原文件。
2. **Git 预准备**：准备对写入变动执行版本锁定的提交元信息。

#### 第五步：安全更新与本地验证门 (Safe Update & Dry-Run Gate)
1. **本地验证门 (Dry-Run Gate)**：
   - 自动识别当前 workspace 根目录的测试环境配置文件（如 `package.json`, `Cargo.toml`, `requirements.txt`, `go.mod`, `Makefile` 等）。
   - 自动在终端执行项目标准的非交互式测试命令（例如 `npm run test`, `pytest`, `cargo test` 或 `go test ./...`）。
   - **拦截与归档逻辑**：如果测试命令运行失败（Exit Code $\ne 0$），则认为新总结的规则规约中存在导致系统崩溃或功能破坏的潜在危害（例如强制修改的代码引发了运行时错误）。此时必须中止物理写入流程，向用户发出警报，列出测试报错日志，并将该条被拦截的规则降级归档到 `AGENTS.md` 的 `<!-- rule-evolution:rejected-start/end -->` 区域中以备忘。
2. **分层写入与清理**：若测试通过或当前项目无任何测试套件，精确定位至以下三层锚点对之一写入：
   - **核心规约层**：`<!-- rule-evolution:core-start -->` / `<!-- rule-evolution:core-end -->`
   - **草案孵化层**：`<!-- rule-evolution:draft-start -->` / `<!-- rule-evolution:draft-end -->`
   - **个人偏好层**：`<!-- rule-evolution:user-prefs-start -->` / `<!-- rule-evolution:user-prefs-end -->`
   写入时在行尾追加带 `hits`（初始为1）、`created` 和 `session` 的标准元数据标记。验证写入结果无误后，删除临时备份 `AGENTS.md.bak`。
3. **Git 提交确认**：在终端执行 Git 提交，对规则更新进行版本锁定：
   ```bash
   git add <agents.md_path> && git commit -m "chore(agents): 自动规则沉淀 - [简短规则主题描述]"
   ```

#### 第六步：草稿晋升、拒绝历史与技能分流 (Promotion, Rejection & Graduation)
1. **Hits 命中统计**：在每次会话启动、AI 读取 `AGENTS.md` 时，如果 AI 判定某条 Draft（草稿层）或 User Prefs（偏好层）规则对本次代码改动有直接的指导或避免了报错，则在沉淀更新时，顺便将该规则行尾的 `hits:N` 计数器自增 1。
2. **拒绝历史归档**：
   - 如果 AI 代理在阶段一发起 Proposal，而用户明确选择“拒绝/忽略”时，AI 代理应当在第二阶段自动把该条拟沉淀的规则（含核心词和简要摘要）写入 [AGENTS.md](file:///Users/cheng/.gemini/config/AGENTS.md) 的拒绝记录区 `<!-- rule-evolution:rejected-start/end -->` 中，作为以后的防打扰过滤依据。
3. **自动晋升核心 (Promotion)**：当某条草稿层规则的 `hits` 达到 3 次时，AI 代理应当在会话结束时，主动提示用户将其从草稿层晋升到核心规约层（Core Rules）或分类主板块中。
4. **技能打包分流 (Graduation & Split)**：
   - 当草案区或偏好区中，针对**同一垂直领域**的条目积累到 3 条及以上（例如：均为关于 Clash 代理分流、或均为关于 Git Commit 规范的细节）且 Hits 累积较高时，AI 代理应触发“细胞分裂”。
   - AI 会建议用户将其打包为独立的 Skill，在获得确认后，AI 会在全局技能路径 `skills/` 下自动新建一个垂直子目录（如 `skills/clash-rules/`），生成符合 `agentskills.io` 规范的 `SKILL.md`，并将这些条目迁移过去，最后在 `AGENTS.md` 中擦除对应草案以防臃肿。
