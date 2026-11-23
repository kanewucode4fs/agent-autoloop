# 现有系统基线追溯计划（事后整理）

> **给执行者**：本文件是对已实现系统的任务与代码映射，不是让 Agent 重做功能的待实施计划。不重演历史 TDD、不伪造提交顺序，也不默认启动子 Agent。本次只补资料；未来真要写新代码时，另建经批准的计划并按 superpowers:executing-plans 或 superpowers:subagent-driven-development 执行。

**目标**：按用户明确授权，把已有需求—设计—任务—实现—证据对应起来，使后续交付检查有完整输入。

**架构**：沿用被动内核、Node 外壳、worker、通道与产品插件。判断与日志投影同事务；外部程序使用 HTTP，不新增服务、依赖、表、接口或页面。

**技术栈**：Node.js >=24、TypeScript、node:sqlite、node:test；TypeScript 与 Prettier 是现有开发工具，无运行时 npm 包。

**Spec**：[现有系统基线设计](../specs/2026-11-23-existing-system-baseline-design.md)。

**实现基准**：`71cc1d24f020ef9d9de9f1b395e367e3452dd61a`；阶段 iteration；整理日期 2026-11-23。

## 全局约束

- 内核只收、只分、只派、只记；不发外部请求、不读取时钟或随机数、不认识具体产品。
- Node.js >=24；package.json 不增加 dependencies。
- decide 只产生本次事实；commit 独占日志追加；evolve 独占持久投影写入。
- 日志、投影、游标及索引同事务；升级不写数据迁移，重放不重新执行业务。
- 外部正文为资料；Agent、cwd、权限、插件和交付仓库来自可信规则及项目 main。
- 一轮不等人；提问结束本轮，回答后再处理。
- 插件统一 `plugins/<name>/{agent,channel,source}.ts`，数据根插件优先。
- 保持 /v1、规则、插件、命令及旧数据兼容；版本号只写 tag。
- 新功能或 bug 才执行真实 TDD；不为事后整理假造红灯、历史提交或完成记录。
- 不运行真实交付脚本、不发真实消息、不重复开启在线飞书监听。
- 性能只记录实测；不设发布达标线，不批量化持久提交换数字。

## 状态读法

- `[x] 定位`：已在固定基准代码找到对应入口和现有测试，不表示历史上有过本计划。
- `[ ] 交付复核`：尚需把新资料提交到固定快照并完成正式一致性检查，不表示要重新实现功能。
- 各任务的“复现命令”供需要时运行，**不是本轮已经执行的命令清单**。当前验证事实见本文末尾，不将所有任务勾成实测通过。
- 共享文件按列出的函数/职责区分；一个文件可以支持多个任务，不因同名文件就认定一个事项。
- 未来页面为 B1/FUTURE-1，留在需求待办；没有实现任务或已完成代码 ID。

## 复核重点

| 输入或失败方式 | 对应任务 | 已有测试入口 | 仍需注意 |
| --- | --- | --- | --- |
| 外部正文试图改变指令、目录或权限 | TASK-2、13 | contract、http、purity | 资料隔离不是所有提示注入的绝对防线 |
| 重复事件、丢失回执响应或回答竞争 | TASK-1、6、8 | events、cards、kernel、stream | 外部业务副作用仍需自行幂等 |
| 已接回会话后断开、进程查询失败 | TASK-5、7、10 | acp、worker、worker-lookup、resume-thread | 不能误开新会话重做，unknown 不等于完成 |
| PR head/main 变化、草稿检查或 Ship/健康不同步 | TASK-15 | delivery、delivery-run、delivery-action | 假 GitHub 测试不证明真实仓库交付已验收 |
| Windows 进程树、登录任务与真实 CLI 工具 | TASK-10、14 | windows、windows-service、worker-login-shell | 本机是 macOS；Windows / fish 有明确跳过 |

以上风险都有已有规格测试，不为本次补文档另写镜像测试；仍未实际运行的环境保留待验收。

## 事项与 ID 映射

B1 为整套当前实现基线，任务按可独立核对的行为域拆分。每条原需求的独立验收 ID 见 spec 的 78 行表，不重复改变产品需求编号。

| 任务 | 需求 ID | 设计节 | 实现组 |
| --- | --- | --- | --- |
| B1/TASK-1 事件接入与来源结果 | B1/REQ-1、B1/REQ-2、B1/REQ-3 | B1/PLAN-§1 | B1/CODE-A |
| B1/TASK-2 可信规则与指令资料分离 | B1/REQ-4、B1/REQ-5、B1/REQ-6、B1/REQ-7 | B1/PLAN-§2 | B1/CODE-B |
| B1/TASK-3 定时、时区与到期 | B1/REQ-8、B1/REQ-9、B1/REQ-10、B1/REQ-11 | B1/PLAN-§3 | B1/CODE-C |
| B1/TASK-4 事项、合并和并发领取 | B1/REQ-12、B1/REQ-13、B1/REQ-14、B1/REQ-15 | B1/PLAN-§4 | B1/CODE-D |
| B1/TASK-5 Agent 插件与进程结果读取 | B1/REQ-16、B1/REQ-17、B1/REQ-18、B1/REQ-19 | B1/PLAN-§5 | B1/CODE-E |
| B1/TASK-6 卡片创建、投递和本人回答 | B1/REQ-20、B1/REQ-21、B1/REQ-22、B1/REQ-23 | B1/PLAN-§6 | B1/CODE-F |
| B1/TASK-7 回答续办与会话归属 | B1/REQ-24、B1/REQ-25、B1/REQ-26、B1/REQ-27 | B1/PLAN-§7 | B1/CODE-G |
| B1/TASK-8 回执、重放与后续动作 | B1/REQ-28、B1/REQ-29、B1/REQ-30、B1/REQ-31 | B1/PLAN-§8 | B1/CODE-H |
| B1/TASK-9 失败重派、额度与停止 | B1/REQ-32、B1/REQ-33、B1/REQ-34、B1/REQ-35、B1/REQ-36、B1/REQ-37 | B1/PLAN-§9 | B1/CODE-I |
| B1/TASK-10 本地执行记录、恢复和清理 | B1/REQ-38、B1/REQ-39、B1/REQ-40、B1/REQ-41、B1/REQ-42 | B1/PLAN-§10 | B1/CODE-J |
| B1/TASK-11 本人状态、历史、事实流和用量 | B1/REQ-43、B1/REQ-44、B1/REQ-45、B1/REQ-46、B1/REQ-47、B1/REQ-75 | B1/PLAN-§11 | B1/CODE-K |
| B1/TASK-12 暂停、恢复、补跑和手动触发 | B1/REQ-48、B1/REQ-49、B1/REQ-50、B1/REQ-51、B1/REQ-52 | B1/PLAN-§12 | B1/CODE-L |
| B1/TASK-13 协议、身份、MCP 与统一插件入口 | B1/REQ-53、B1/REQ-54、B1/REQ-55、B1/REQ-56、B1/REQ-57、B1/REQ-74 | B1/PLAN-§13 | B1/CODE-M |
| B1/TASK-14 安装、常驻、多机、备份及维护 | B1/REQ-58、B1/REQ-59、B1/REQ-60、B1/REQ-61、B1/REQ-62、B1/REQ-63、B1/REQ-77 | B1/PLAN-§14 | B1/CODE-N |
| B1/TASK-15 GitHub PR 交付插件 | B1/REQ-64、B1/REQ-65、B1/REQ-66、B1/REQ-67、B1/REQ-68 | B1/PLAN-§15 | B1/CODE-O |
| B1/TASK-16 场景模板与浏览器交接 | B1/REQ-69、B1/REQ-70、B1/REQ-71、B1/REQ-72 | B1/PLAN-§16 | B1/CODE-P |
| B1/TASK-17 日志投影、兼容性与工程验证 | B1/REQ-73、B1/REQ-76、B1/REQ-78 | B1/PLAN-§17 | B1/CODE-Q |
| B1/TASK-18 文档基线与追溯入口 | B1/REQ-75 | B1/PLAN-§18 | B1/CODE-R |

TASK-17 为非功能底座和验证配套；TASK-18 为本次资料补齐，支持可追溯需求及用户当前授权。它们不是额外的产品能力。

## 逐项追溯任务

### TASK-1 事件接入与来源结果

**需求与设计**：[产品 REQ-1](../../01-需求文档/需求.md)；B1/REQ-1、B1/REQ-2、B1/REQ-3；[B1/PLAN-§1](../specs/2026-11-23-existing-system-baseline-design.md#plan-1-事件接入与来源结果)。
**已存在文件 / 职责**：[kernel/decide.ts](../../../kernel/decide.ts)、[kernel/query.ts](../../../kernel/query.ts)、[protocol/validate.ts](../../../protocol/validate.ts)。
**输入**：已认证来源的 EventInput，由 parseEvent 校验；时间由 host 注入。
**输出 / 实现组**：B1/CODE-A，`ingest`、`recordEvent`、`eventOutcome`、`parseEvent` 的现有职责；不产生本轮新的功能实现。

- [x] **定位实现**：在上述基准确认入口及职责，沿现有依赖调用；来源身份来自请求凭据，id 为来源内稳定编号。ingest 校验后匹配可信规则；未匹配、暂停跳过和超过 hops 的事件仍按协议留痕。查询结果须防止泄露混入其他来源的资料。
- [x] **定位验证依据**：[tests/events.test.ts](../../../tests/events.test.ts)、[tests/event-outcome.test.ts](../../../tests/event-outcome.test.ts)、[tests/hops.test.ts](../../../tests/hops.test.ts)；核对断言：重复事件只有一个 event/派发；另一来源的事件返回 404；hops 超过 8 留记录、不继续派发。
- [ ] **固定提交后做交付复核**：对本任务检查需求→设计、设计→任务、任务→代码及三个逆向映射；若发现差距先记录，不在本追溯任务内擅自改代码。

**复现命令（按需，不代表本次已跑）**：`node --disable-warning=ExperimentalWarning --test tests/events.test.ts tests/event-outcome.test.ts tests/hops.test.ts`。
**通过口径**：对应行为测试无失败；平台跳过单列，不据此宣称该平台已实测。

### TASK-2 可信规则与指令资料分离

**需求与设计**：[产品 REQ-2](../../01-需求文档/需求.md)；B1/REQ-4、B1/REQ-5、B1/REQ-6、B1/REQ-7；[B1/PLAN-§2](../specs/2026-11-23-existing-system-baseline-design.md#plan-2-可信规则与指令资料分离)。
**已存在文件 / 职责**：[protocol/rules.ts](../../../protocol/rules.ts)、[kernel/rules.ts](../../../kernel/rules.ts)、[worker/contract.ts](../../../worker/contract.ts)、[host/runtime.ts](../../../host/runtime.ts)。
**输入**：本人维护的规则与可信任务说明；事件 context 仅为资料。
**输出 / 实现组**：B1/CODE-B，`parseRules`、`promptFor`、`reloadRules` 的现有职责；不产生本轮新的功能实现。

- [x] **定位实现**：在上述基准确认入口及职责，沿现有依赖调用；规则校验与类型/匹配分层；规则和 Skill 由本人维护。执行提示词只引用资料文件，业务文本不获得路由或权限。热加载错误保留旧配置；首次没有有效规则则不启动。
- [x] **定位验证依据**：[tests/rules.test.ts](../../../tests/rules.test.ts)、[tests/contract.test.ts](../../../tests/contract.test.ts)、[tests/rules-preview.test.ts](../../../tests/rules-preview.test.ts)；核对断言：坏引用拒收；prompt 不含事件正文；explain/dry-run 不写中台、不泄露私有任务说明。
- [ ] **固定提交后做交付复核**：对本任务检查需求→设计、设计→任务、任务→代码及三个逆向映射；若发现差距先记录，不在本追溯任务内擅自改代码。

**复现命令（按需，不代表本次已跑）**：`node --disable-warning=ExperimentalWarning --test tests/rules.test.ts tests/contract.test.ts tests/rules-preview.test.ts`。
**通过口径**：对应行为测试无失败；平台跳过单列，不据此宣称该平台已实测。

### TASK-3 定时、时区与到期

**需求与设计**：[产品 REQ-3](../../01-需求文档/需求.md)；B1/REQ-8、B1/REQ-9、B1/REQ-10、B1/REQ-11；[B1/PLAN-§3](../specs/2026-11-23-existing-system-baseline-design.md#plan-3-定时时区与到期)。
**已存在文件 / 职责**：[kernel/cron.ts](../../../kernel/cron.ts)、[kernel/due.ts](../../../kernel/due.ts)、[kernel/decide.ts](../../../kernel/decide.ts)。
**输入**：沿用 spec 的当前接口契约和可信配置；不新增相邻模块 API。
**输出 / 实现组**：B1/CODE-C，`parseCron`、`nextRun`、`latestBefore`、`deadlines`、`decide` 的现有职责；不产生本轮新的功能实现。

- [x] **定位实现**：在上述基准确认入口及职责，沿现有依赖调用；host 注入时间；cron、复查、退避、租约及离线判断共用 due 索引和外壳单一定时器。规则或时区变化重新计算计划。心跳记录受一分钟节流，不在每次轮询追加事实。
- [x] **定位验证依据**：[tests/cron.test.ts](../../../tests/cron.test.ts)、[tests/schedule.test.ts](../../../tests/schedule.test.ts)、[tests/heartbeat.test.ts](../../../tests/heartbeat.test.ts)；核对断言：固定注入时间决定触发；时钟恢复只补最近时刻；夏令时重复取第一次、不存在的时刻跳过。
- [ ] **固定提交后做交付复核**：对本任务检查需求→设计、设计→任务、任务→代码及三个逆向映射；若发现差距先记录，不在本追溯任务内擅自改代码。

**复现命令（按需，不代表本次已跑）**：`node --disable-warning=ExperimentalWarning --test tests/cron.test.ts tests/schedule.test.ts tests/heartbeat.test.ts`。
**通过口径**：对应行为测试无失败；平台跳过单列，不据此宣称该平台已实测。

### TASK-4 事项、合并和并发领取

**需求与设计**：[产品 REQ-4](../../01-需求文档/需求.md)；B1/REQ-12、B1/REQ-13、B1/REQ-14、B1/REQ-15；[B1/PLAN-§4](../specs/2026-11-23-existing-system-baseline-design.md#plan-4-事项合并和并发领取)。
**已存在文件 / 职责**：[kernel/decide.ts](../../../kernel/decide.ts)、[kernel/query.ts](../../../kernel/query.ts)、[worker/worker.ts](../../../worker/worker.ts)、[protocol/job.ts](../../../protocol/job.ts)。
**输入**：沿用 spec 的当前接口契约和可信配置；不新增相邻模块 API。
**输出 / 实现组**：B1/CODE-D，`enqueue`、`claimNext`、`assignQueued`、`jobOf`、`createWorker` 的现有职责；不产生本轮新的功能实现。

- [x] **定位实现**：在上述基准确认入口及职责，沿现有依赖调用；事项以 rule+lane 表示，没有独立任务状态表。同事项排队事件合并；在跑期间的新事件排后续轮。领取时采用当前规则，领取后保持执行归属；重复 claim 仍拿同一工作但更新能力 token。
- [x] **定位验证依据**：[tests/runs.test.ts](../../../tests/runs.test.ts)、[tests/queue-costs.test.ts](../../../tests/queue-costs.test.ts)、[tests/worker.test.ts](../../../tests/worker.test.ts)；核对断言：同 lane 不并发；同 claim 不重复派活；多层限额均受测试约束；worker 默认 capacity=2。
- [ ] **固定提交后做交付复核**：对本任务检查需求→设计、设计→任务、任务→代码及三个逆向映射；若发现差距先记录，不在本追溯任务内擅自改代码。

**复现命令（按需，不代表本次已跑）**：`node --disable-warning=ExperimentalWarning --test tests/runs.test.ts tests/queue-costs.test.ts tests/worker.test.ts`。
**通过口径**：对应行为测试无失败；平台跳过单列，不据此宣称该平台已实测。

### TASK-5 Agent 插件与进程结果读取

**需求与设计**：[产品 REQ-5](../../01-需求文档/需求.md)；B1/REQ-16、B1/REQ-17、B1/REQ-18、B1/REQ-19；[B1/PLAN-§5](../specs/2026-11-23-existing-system-baseline-design.md#plan-5-agent-插件与进程结果读取)。
**已存在文件 / 职责**：[worker/agents.ts](../../../worker/agents.ts)、[worker/worker.ts](../../../worker/worker.ts)、[plugins/command/agent.ts](../../../plugins/command/agent.ts)、[plugins/claude/agent.ts](../../../plugins/claude/agent.ts)、[plugins/codex/agent.ts](../../../plugins/codex/agent.ts)、[plugins/acp/agent.ts](../../../plugins/acp/agent.ts)、[plugins/acp/bridge.ts](../../../plugins/acp/bridge.ts)、[plugins/dsh/agent.ts](../../../plugins/dsh/agent.ts)、[plugins/zcode/agent.ts](../../../plugins/zcode/agent.ts)。
**输入**：沿用 spec 的当前接口契约和可信配置；不新增相邻模块 API。
**输出 / 实现组**：B1/CODE-E，`adapterFor`、`findProgram`、`createWorker`、`launch`、`conclude` 的现有职责；不产生本轮新的功能实现。

- [x] **定位实现**：在上述基准确认入口及职责，沿现有依赖调用；适配器以 Adapter.command/read/link 描述专用差异；通用 command 只保证进程契约，不替任意程序限制权限。ACP 读协议 v1，会话历史不能变成本轮消息。CLI 自身登录和工具由执行环境提供。
- [x] **定位验证依据**：[tests/adapters.test.ts](../../../tests/adapters.test.ts)、[tests/acp.test.ts](../../../tests/acp.test.ts)、[tests/dsh.test.ts](../../../tests/dsh.test.ts)、[tests/zcode.test.ts](../../../tests/zcode.test.ts)、[tests/worker-path.test.ts](../../../tests/worker-path.test.ts)；核对断言：参数含空格/特殊字符仍完整；未结束输出不算完成；恢复会话与权限模式按适配器逐项验证。
- [ ] **固定提交后做交付复核**：对本任务检查需求→设计、设计→任务、任务→代码及三个逆向映射；若发现差距先记录，不在本追溯任务内擅自改代码。

**复现命令（按需，不代表本次已跑）**：`node --disable-warning=ExperimentalWarning --test tests/adapters.test.ts tests/acp.test.ts tests/dsh.test.ts tests/zcode.test.ts tests/worker-path.test.ts`。
**通过口径**：对应行为测试无失败；平台跳过单列，不据此宣称该平台已实测。

### TASK-6 卡片创建、投递和本人回答

**需求与设计**：[产品 REQ-6](../../01-需求文档/需求.md)；B1/REQ-20、B1/REQ-21、B1/REQ-22、B1/REQ-23；[B1/PLAN-§6](../specs/2026-11-23-existing-system-baseline-design.md#plan-6-卡片创建投递和本人回答)。
**已存在文件 / 职责**：[kernel/decide.ts](../../../kernel/decide.ts)、[kernel/card-page.ts](../../../kernel/card-page.ts)、[host/channels.ts](../../../host/channels.ts)、[plugins/desktop/channel.ts](../../../plugins/desktop/channel.ts)、[plugins/feishu/channel.ts](../../../plugins/feishu/channel.ts)、[plugins/webhook/channel.ts](../../../plugins/webhook/channel.ts)、[plugins/webhook/templates.ts](../../../plugins/webhook/templates.ts)。
**输入**：沿用 spec 的当前接口契约和可信配置；不新增相邻模块 API。
**输出 / 实现组**：B1/CODE-F，`createCard`、`nextCard`、`answer`、`closeCard` 的现有职责；不产生本轮新的功能实现。

- [x] **定位实现**：在上述基准确认入口及职责，沿现有依赖调用；内容创建后不变，版本只表达状态变更。提问 open→decided→done，直接结案 open→closed；FYI 创建即 closed。通道报告版本和 ref；卡片表单转义外来内容、不执行脚本，GET 不改变决定。
- [x] **定位验证依据**：[tests/cards.test.ts](../../../tests/cards.test.ts)、[tests/channels.test.ts](../../../tests/channels.test.ts)、[tests/desktop.test.ts](../../../tests/desktop.test.ts)、[tests/webhook-templates.test.ts](../../../tests/webhook-templates.test.ts)；核对断言：重复问题保持原卡；越权回答被拒；链接从创建起 7 天有效；假通知记录一次，不发真消息。
- [ ] **固定提交后做交付复核**：对本任务检查需求→设计、设计→任务、任务→代码及三个逆向映射；若发现差距先记录，不在本追溯任务内擅自改代码。

**复现命令（按需，不代表本次已跑）**：`node --disable-warning=ExperimentalWarning --test tests/cards.test.ts tests/channels.test.ts tests/desktop.test.ts tests/webhook-templates.test.ts`。
**通过口径**：对应行为测试无失败；平台跳过单列，不据此宣称该平台已实测。

### TASK-7 回答续办与会话归属

**需求与设计**：[产品 REQ-7](../../01-需求文档/需求.md)；B1/REQ-24、B1/REQ-25、B1/REQ-26、B1/REQ-27；[B1/PLAN-§7](../specs/2026-11-23-existing-system-baseline-design.md#plan-7-回答续办与会话归属)。
**已存在文件 / 职责**：[kernel/decide.ts](../../../kernel/decide.ts)、[kernel/query.ts](../../../kernel/query.ts)、[worker/worker.ts](../../../worker/worker.ts)。
**输入**：沿用 spec 的当前接口契约和可信配置；不新增相邻模块 API。
**输出 / 实现组**：B1/CODE-G，`answer`、`settleSession`、`applyReceipt`、`askedAbout`、`jobOf` 的现有职责；不产生本轮新的功能实现。

- [x] **定位实现**：在上述基准确认入口及职责，沿现有依赖调用；提问结束本轮，不阻塞 Agent 等人。回答和复查携带原 lane、Agent 及会话归属；会话冲突时保留资料并开新会话。elsewhere 续办使用配置的任务上下文，不接外部定时任务自己的会话。
- [x] **定位验证依据**：[tests/resume-thread.test.ts](../../../tests/resume-thread.test.ts)、[tests/stops-and-threads.test.ts](../../../tests/stops-and-threads.test.ts)、[tests/loop.test.ts](../../../tests/loop.test.ts)、[tests/elsewhere-ask.test.ts](../../../tests/elsewhere-ask.test.ts)；核对断言：绿色回答回到原事项，卡片从 decided 变 done；跨 Agent 复查不能沿用旧 Agent 会话。
- [ ] **固定提交后做交付复核**：对本任务检查需求→设计、设计→任务、任务→代码及三个逆向映射；若发现差距先记录，不在本追溯任务内擅自改代码。

**复现命令（按需，不代表本次已跑）**：`node --disable-warning=ExperimentalWarning --test tests/resume-thread.test.ts tests/stops-and-threads.test.ts tests/loop.test.ts tests/elsewhere-ask.test.ts`。
**通过口径**：对应行为测试无失败；平台跳过单列，不据此宣称该平台已实测。

### TASK-8 回执、重放与后续动作

**需求与设计**：[产品 REQ-8](../../01-需求文档/需求.md)；B1/REQ-28、B1/REQ-29、B1/REQ-30、B1/REQ-31；[B1/PLAN-§8](../specs/2026-11-23-existing-system-baseline-design.md#plan-8-回执重放与后续动作)。
**已存在文件 / 职责**：[protocol/validate.ts](../../../protocol/validate.ts)、[worker/contract.ts](../../../worker/contract.ts)、[worker/worker.ts](../../../worker/worker.ts)、[kernel/decide.ts](../../../kernel/decide.ts)、[kernel/store.ts](../../../kernel/store.ts)、[kernel/evolve.ts](../../../kernel/evolve.ts)。
**输入**：沿用 spec 的当前接口契约和可信配置；不新增相邻模块 API。
**输出 / 实现组**：B1/CODE-H，`parseResult`、`readMessage`、`resultFrom`、`conclude`、`applyReceipt`、`finish`、`commit`、`evolve` 的现有职责；不产生本轮新的功能实现。

- [x] **定位实现**：在上述基准确认入口及职责，沿现有依赖调用；普通 Agent 输出第一行结论及可选 agent-autoloop 代码块。ask/notice/resolve/later/events 由中台校验范围；failed 不生效提问、结案、复查和新事件。支持会话的修复最多两次，emits 与 hops 不能由回执自定越权。
- [x] **定位验证依据**：[tests/contract.test.ts](../../../tests/contract.test.ts)、[tests/worker.test.ts](../../../tests/worker.test.ts)、[tests/kernel.test.ts](../../../tests/kernel.test.ts)、[tests/stream.test.ts](../../../tests/stream.test.ts)；核对断言：事务失败全部回滚；回执重放确认相同；修复最多 2 次；超过 8 hops 只记不派。
- [ ] **固定提交后做交付复核**：对本任务检查需求→设计、设计→任务、任务→代码及三个逆向映射；若发现差距先记录，不在本追溯任务内擅自改代码。

**复现命令（按需，不代表本次已跑）**：`node --disable-warning=ExperimentalWarning --test tests/contract.test.ts tests/worker.test.ts tests/kernel.test.ts tests/stream.test.ts`。
**通过口径**：对应行为测试无失败；平台跳过单列，不据此宣称该平台已实测。

### TASK-9 失败重派、额度与停止

**需求与设计**：[产品 REQ-9](../../01-需求文档/需求.md)；B1/REQ-32、B1/REQ-33、B1/REQ-34、B1/REQ-35、B1/REQ-36、B1/REQ-37；[B1/PLAN-§9](../specs/2026-11-23-existing-system-baseline-design.md#plan-9-失败重派额度与停止)。
**已存在文件 / 职责**：[kernel/decide.ts](../../../kernel/decide.ts)、[kernel/due.ts](../../../kernel/due.ts)、[worker/worker.ts](../../../worker/worker.ts)。
**输入**：沿用 spec 的当前接口契约和可信配置；不新增相邻模块 API。
**输出 / 实现组**：B1/CODE-I，`retryLater`、`limit`、`lift`、`sweep`、`stopRun`、`finish` 的现有职责；不产生本轮新的功能实现。

- [x] **定位实现**：在上述基准确认入口及职责，沿现有依赖调用；重试属于中台；worker 不建立另一套业务重试。limitedUntil 必须是有效窗口内的带时区时间。需要本人恢复登录、连接或验证应 done+ask；unknown 不等于失败，迟到结果仍按能力核验接收。
- [x] **定位验证依据**：[tests/runs.test.ts](../../../tests/runs.test.ts)、[tests/usage-limit.test.ts](../../../tests/usage-limit.test.ts)、[tests/stops-and-threads.test.ts](../../../tests/stops-and-threads.test.ts)、[tests/heartbeat.test.ts](../../../tests/heartbeat.test.ts)；核对断言：失败轮次 attempts/notBefore 正确；额度恢复后继续；停止中失败不重派。
- [ ] **固定提交后做交付复核**：对本任务检查需求→设计、设计→任务、任务→代码及三个逆向映射；若发现差距先记录，不在本追溯任务内擅自改代码。

**复现命令（按需，不代表本次已跑）**：`node --disable-warning=ExperimentalWarning --test tests/runs.test.ts tests/usage-limit.test.ts tests/stops-and-threads.test.ts tests/heartbeat.test.ts`。
**通过口径**：对应行为测试无失败；平台跳过单列，不据此宣称该平台已实测。

### TASK-10 本地执行记录、恢复和清理

**需求与设计**：[产品 REQ-10](../../01-需求文档/需求.md)；B1/REQ-38、B1/REQ-39、B1/REQ-40、B1/REQ-41、B1/REQ-42；[B1/PLAN-§10](../specs/2026-11-23-existing-system-baseline-design.md#plan-10-本地执行记录恢复和清理)。
**已存在文件 / 职责**：[worker/worker.ts](../../../worker/worker.ts)、[worker/run.ts](../../../worker/run.ts)、[worker/windows.ps1](../../../worker/windows.ps1)、[worker/usage.ts](../../../worker/usage.ts)、[host/command.ts](../../../host/command.ts)、[host/node-path.ts](../../../host/node-path.ts)。
**输入**：沿用 spec 的当前接口契约和可信配置；不新增相邻模块 API。
**输出 / 实现组**：B1/CODE-J，`createWorker`、`recoverEntry`、`launch`、`endWith`、`settle`、`tidy` 的现有职责；不产生本轮新的功能实现。

- [x] **定位实现**：在上述基准确认入口及职责，沿现有依赖调用；entry 和结果先持久化，再启动或提交。Unix 跟踪带标记进程组，Windows 通过监督进程和 Job Object 管理后代。默认超时 180 分钟，最终消息后留 30 秒退出，停止宽限 10 秒，完成输出保留 14 天。
- [x] **定位验证依据**：[tests/worker.test.ts](../../../tests/worker.test.ts)、[tests/worker-lookup.test.ts](../../../tests/worker-lookup.test.ts)、[tests/worker-active.test.ts](../../../tests/worker-active.test.ts)、[tests/worker-tidy.test.ts](../../../tests/worker-tidy.test.ts)、[tests/session-usage.test.ts](../../../tests/session-usage.test.ts)、[tests/windows.test.ts](../../../tests/windows.test.ts)；核对断言：查询失败保留运行状态；恢复不重复业务；缺可信用量基线不冒报累计或零；有 entry 的目录不被清理。
- [ ] **固定提交后做交付复核**：对本任务检查需求→设计、设计→任务、任务→代码及三个逆向映射；若发现差距先记录，不在本追溯任务内擅自改代码。

**复现命令（按需，不代表本次已跑）**：`node --disable-warning=ExperimentalWarning --test tests/worker.test.ts tests/worker-lookup.test.ts tests/worker-active.test.ts tests/worker-tidy.test.ts tests/session-usage.test.ts tests/windows.test.ts`。
**通过口径**：对应行为测试无失败；平台跳过单列，不据此宣称该平台已实测。

### TASK-11 本人状态、历史、事实流和用量

**需求与设计**：[产品 REQ-11](../../01-需求文档/需求.md)；B1/REQ-43、B1/REQ-44、B1/REQ-45、B1/REQ-46、B1/REQ-47、B1/REQ-75；[B1/PLAN-§11](../specs/2026-11-23-existing-system-baseline-design.md#plan-11-本人状态历史事实流和用量)。
**已存在文件 / 职责**：[host/state.ts](../../../host/state.ts)、[host/runtime.ts](../../../host/runtime.ts)、[cli/status.ts](../../../cli/status.ts)、[cli/runs.ts](../../../cli/runs.ts)、[cli/why.ts](../../../cli/why.ts)、[cli/spend.ts](../../../cli/spend.ts)、[protocol/client.ts](../../../protocol/client.ts)、[kernel/query.ts](../../../kernel/query.ts)。
**输入**：沿用 spec 的当前接口契约和可信配置；不新增相邻模块 API。
**输出 / 实现组**：B1/CODE-K，`stateOf`、`ruleHistory`、`spendingOf`、`why`、`logAfter`、`safeFact` 的现有职责；不产生本轮新的功能实现。

- [x] **定位实现**：在上述基准确认入口及职责，沿现有依赖调用；host 查询只读视图，不在查询中作业务决定。本人先取 state.seq，再从 stream after 续读；断线可用 Last-Event-ID。快照默认最近 60 轮，CLI runs 每页 20 条，统计不随历史分页截断。
- [x] **定位验证依据**：[tests/status-json.test.ts](../../../tests/status-json.test.ts)、[tests/status-view.test.ts](../../../tests/status-view.test.ts)、[tests/status-cost.test.ts](../../../tests/status-cost.test.ts)、[tests/cli-runs.test.ts](../../../tests/cli-runs.test.ts)、[tests/rule-history.test.ts](../../../tests/rule-history.test.ts)、[tests/why.test.ts](../../../tests/why.test.ts)、[tests/stream.test.ts](../../../tests/stream.test.ts)；核对断言：快照与 SSE 序号衔接；历史分页完整；流水不泄露密钥哈希；管道无颜色污染。
- [ ] **固定提交后做交付复核**：对本任务检查需求→设计、设计→任务、任务→代码及三个逆向映射；若发现差距先记录，不在本追溯任务内擅自改代码。

**复现命令（按需，不代表本次已跑）**：`node --disable-warning=ExperimentalWarning --test tests/status-json.test.ts tests/status-view.test.ts tests/status-cost.test.ts tests/cli-runs.test.ts tests/rule-history.test.ts tests/why.test.ts tests/stream.test.ts`。
**通过口径**：对应行为测试无失败；平台跳过单列，不据此宣称该平台已实测。

### TASK-12 暂停、恢复、补跑和手动触发

**需求与设计**：[产品 REQ-12](../../01-需求文档/需求.md)；B1/REQ-48、B1/REQ-49、B1/REQ-50、B1/REQ-51、B1/REQ-52；[B1/PLAN-§12](../specs/2026-11-23-existing-system-baseline-design.md#plan-12-暂停恢复补跑和手动触发)。
**已存在文件 / 职责**：[kernel/decide.ts](../../../kernel/decide.ts)、[kernel/query.ts](../../../kernel/query.ts)、[host/state.ts](../../../host/state.ts)、[cli/main.ts](../../../cli/main.ts)。
**输入**：沿用 spec 的当前接口契约和可信配置；不新增相邻模块 API。
**输出 / 实现组**：B1/CODE-L，`decide`、`pauseOf`、`skippedOf` 的现有职责；不产生本轮新的功能实现。

- [x] **定位实现**：在上述基准确认入口及职责，沿现有依赖调用；pauseRule/resumeRule/replayRule/runRule 是 decide 的控制输入，在日志内表达，不改 rules.json。恢复应答携带原暂停和跳过概况；until 到期自动恢复。stop 仍是轮次级操作。
- [x] **定位验证依据**：[tests/pause.test.ts](../../../tests/pause.test.ts)、[tests/pause-http.test.ts](../../../tests/pause-http.test.ts)；核对断言：控制操作只接受 owner；暂停 run/replay 返回 409；恢复给实际 since 以供本人决定是否补跑。
- [ ] **固定提交后做交付复核**：对本任务检查需求→设计、设计→任务、任务→代码及三个逆向映射；若发现差距先记录，不在本追溯任务内擅自改代码。

**复现命令（按需，不代表本次已跑）**：`node --disable-warning=ExperimentalWarning --test tests/pause.test.ts tests/pause-http.test.ts`。
**通过口径**：对应行为测试无失败；平台跳过单列，不据此宣称该平台已实测。

### TASK-13 协议、身份、MCP 与统一插件入口

**需求与设计**：[产品 REQ-13](../../01-需求文档/需求.md)；B1/REQ-53、B1/REQ-54、B1/REQ-55、B1/REQ-56、B1/REQ-57、B1/REQ-74；[B1/PLAN-§13](../specs/2026-11-23-existing-system-baseline-design.md#plan-13-协议身份mcp-与统一插件入口)。
**已存在文件 / 职责**：[kernel/http.ts](../../../kernel/http.ts)、[kernel/credentials.ts](../../../kernel/credentials.ts)、[kernel/errors.ts](../../../kernel/errors.ts)、[protocol/client.ts](../../../protocol/client.ts)、[protocol/schema.ts](../../../protocol/schema.ts)、[protocol/schema.json](../../../protocol/schema.json)、[protocol/conformance.ts](../../../protocol/conformance.ts)、[host/http.ts](../../../host/http.ts)、[host/server.ts](../../../host/server.ts)、[host/plugins.ts](../../../host/plugins.ts)、[host/lines.ts](../../../host/lines.ts)、[cli/mcp.ts](../../../cli/mcp.ts)、[plugins/github/source.ts](../../../plugins/github/source.ts)、[plugins/gitlab/source.ts](../../../plugins/gitlab/source.ts)。
**输入**：沿用 spec 的当前接口契约和可信配置；不新增相邻模块 API。
**输出 / 实现组**：B1/CODE-M，`createHandler`、`whose`、`credential`、`hubClient`、`checkSchema`、`conformance`、`serve`、`makeHandler`、`loadPlugin` 的现有职责；不产生本轮新的功能实现。

- [x] **定位实现**：在上述基准确认入口及职责，沿现有依赖调用；HTTP 路由在 kernel，Node 转换和连接在 host。健康接口仅公开 ok/commit。MCP 使用换行 JSON-RPC 和 2025-11-25 握手；同一协议被 CLI、worker 和通道使用。插件只能以合法名字发现，来源验签秘密在外部插件端。
- [x] **定位验证依据**：[tests/http.test.ts](../../../tests/http.test.ts)、[tests/conformance.test.ts](../../../tests/conformance.test.ts)、[tests/schema.test.ts](../../../tests/schema.test.ts)、[tests/key-remove.test.ts](../../../tests/key-remove.test.ts)、[tests/mcp.test.ts](../../../tests/mcp.test.ts)、[tests/plugins.test.ts](../../../tests/plugins.test.ts)、[tests/github.test.ts](../../../tests/github.test.ts)、[tests/gitlab.test.ts](../../../tests/gitlab.test.ts)、[tests/lines.test.ts](../../../tests/lines.test.ts)；核对断言：错误种类密钥被拒；删除密钥使等待请求失效；来源 HMAC/token 不正确时拒收；Schema 与运行边界一致。
- [ ] **固定提交后做交付复核**：对本任务检查需求→设计、设计→任务、任务→代码及三个逆向映射；若发现差距先记录，不在本追溯任务内擅自改代码。

**复现命令（按需，不代表本次已跑）**：`node --disable-warning=ExperimentalWarning --test tests/http.test.ts tests/conformance.test.ts tests/schema.test.ts tests/key-remove.test.ts tests/mcp.test.ts tests/plugins.test.ts tests/github.test.ts tests/gitlab.test.ts tests/lines.test.ts`。
**通过口径**：对应行为测试无失败；平台跳过单列，不据此宣称该平台已实测。

### TASK-14 安装、常驻、多机、备份及维护

**需求与设计**：[产品 REQ-14](../../01-需求文档/需求.md)；B1/REQ-58、B1/REQ-59、B1/REQ-60、B1/REQ-61、B1/REQ-62、B1/REQ-63、B1/REQ-77；[B1/PLAN-§14](../specs/2026-11-23-existing-system-baseline-design.md#plan-14-安装常驻多机备份及维护)。
**已存在文件 / 职责**：[install.sh](../../../install.sh)、[install.ps1](../../../install.ps1)、[bin/agent-autoloop.ts](../../../bin/agent-autoloop.ts)、[bin/agent-autoloop.cmd](../../../bin/agent-autoloop.cmd)、[cli/main.ts](../../../cli/main.ts)、[cli/update.ts](../../../cli/update.ts)、[cli/backup.ts](../../../cli/backup.ts)、[cli/uninstall.ts](../../../cli/uninstall.ts)、[cli/try.ts](../../../cli/try.ts)、[cli/explain.ts](../../../cli/explain.ts)、[host/up.ts](../../../host/up.ts)、[host/config.ts](../../../host/config.ts)、[host/home.ts](../../../host/home.ts)、[host/files.ts](../../../host/files.ts)、[host/service.ts](../../../host/service.ts)、[host/background.ts](../../../host/background.ts)、[host/background.js](../../../host/background.js)、[host/version.ts](../../../host/version.ts)。
**输入**：沿用 spec 的当前接口契约和可信配置；不新增相邻模块 API。
**输出 / 实现组**：B1/CODE-N，`readConfig`、`writeConfig`、`up`、`installService`、`removeService`、`backup`、`restore`、`updateCode` 的现有职责；不产生本轮新的功能实现。

- [x] **定位实现**：在上述基准确认入口及职责，沿现有依赖调用；默认数据根 ~/.agent-autoloop，默认回环监听 3350；Unix 权限和 Windows ACL 各自限制数据。默认公共源按正式 tag 安装更新，自定义 origin 按其分支快进。主密钥和数据库必须一起备份；没有容器部署或数据库迁移框架。
- [x] **定位验证依据**：[tests/install.test.ts](../../../tests/install.test.ts)、[tests/init-command.test.ts](../../../tests/init-command.test.ts)、[tests/cli-flags.test.ts](../../../tests/cli-flags.test.ts)、[tests/join.test.ts](../../../tests/join.test.ts)、[tests/backup.test.ts](../../../tests/backup.test.ts)、[tests/uninstall.test.ts](../../../tests/uninstall.test.ts)、[tests/releases.test.ts](../../../tests/releases.test.ts)、[tests/service.test.ts](../../../tests/service.test.ts)、[tests/loop.test.ts](../../../tests/loop.test.ts)；核对断言：配置文件私有；help 不产生安装副作用；备份恢复包含 WAL 行为；Windows 原生与登录服务需对应平台另验。
- [ ] **固定提交后做交付复核**：对本任务检查需求→设计、设计→任务、任务→代码及三个逆向映射；若发现差距先记录，不在本追溯任务内擅自改代码。

**复现命令（按需，不代表本次已跑）**：`node --disable-warning=ExperimentalWarning --test tests/install.test.ts tests/init-command.test.ts tests/cli-flags.test.ts tests/join.test.ts tests/backup.test.ts tests/uninstall.test.ts tests/releases.test.ts tests/service.test.ts tests/loop.test.ts`。
**通过口径**：对应行为测试无失败；平台跳过单列，不据此宣称该平台已实测。

### TASK-15 GitHub PR 交付插件

**需求与设计**：[产品 REQ-15](../../01-需求文档/需求.md)；B1/REQ-64、B1/REQ-65、B1/REQ-66、B1/REQ-67、B1/REQ-68；[B1/PLAN-§15](../specs/2026-11-23-existing-system-baseline-design.md#plan-15-github-pr-交付插件)。
**已存在文件 / 职责**：[plugins/delivery/delivery.ts](../../../plugins/delivery/delivery.ts)、[plugins/delivery/run.ts](../../../plugins/delivery/run.ts)、[plugins/delivery/agent.ts](../../../plugins/delivery/agent.ts)、[plugins/delivery/input.ts](../../../plugins/delivery/input.ts)、[plugins/delivery/poll.ts](../../../plugins/delivery/poll.ts)、[plugins/delivery/project.ts](../../../plugins/delivery/project.ts)、[plugins/delivery/review.ts](../../../plugins/delivery/review.ts)、[plugins/delivery/review-ledger.ts](../../../plugins/delivery/review-ledger.ts)、[plugins/delivery/review-policy.ts](../../../plugins/delivery/review-policy.ts)、[plugins/delivery/handoff.ts](../../../plugins/delivery/handoff.ts)、[plugins/delivery/github.ts](../../../plugins/delivery/github.ts)、[plugins/delivery/action.ts](../../../plugins/delivery/action.ts)、[plugins/delivery/action.yml](../../../plugins/delivery/action.yml)。
**输入**：沿用 spec 的当前接口契约和可信配置；不新增相邻模块 API。
**输出 / 实现组**：B1/CODE-O，`deliver`、`poll`、`settings`、`reviewDelivery`、`decideShip` 的现有职责；不产生本轮新的功能实现。

- [x] **定位实现**：在上述基准确认入口及职责，沿现有依赖调用；交付是普通 Agent，不在内核建交付状态机。稳定事件按 PR 分事项；可信提交状态与审查账本绑定轮次和来源。只审查、撤回、退回、仅合并、无需发布和 live 必须区分。真实业务副作用以 GitHub 读回为准。
- [x] **定位验证依据**：[tests/delivery.test.ts](../../../tests/delivery.test.ts)、[tests/delivery-review.test.ts](../../../tests/delivery-review.test.ts)、[tests/delivery-input.test.ts](../../../tests/delivery-input.test.ts)、[tests/delivery-run.test.ts](../../../tests/delivery-run.test.ts)、[tests/delivery-ledger.test.ts](../../../tests/delivery-ledger.test.ts)、[tests/delivery-action.test.ts](../../../tests/delivery-action.test.ts)、[tests/delivery-scope.test.ts](../../../tests/delivery-scope.test.ts)、[tests/delivery-handoff.test.ts](../../../tests/delivery-handoff.test.ts)；核对断言：head/main 变化不误合并；丢失合并响应先读回；只审查不合并；文档后续提交不掩盖未发布代码。
- [ ] **固定提交后做交付复核**：对本任务检查需求→设计、设计→任务、任务→代码及三个逆向映射；若发现差距先记录，不在本追溯任务内擅自改代码。

**复现命令（按需，不代表本次已跑）**：`node --disable-warning=ExperimentalWarning --test tests/delivery.test.ts tests/delivery-review.test.ts tests/delivery-input.test.ts tests/delivery-run.test.ts tests/delivery-ledger.test.ts tests/delivery-action.test.ts tests/delivery-scope.test.ts tests/delivery-handoff.test.ts`。
**通过口径**：对应行为测试无失败；平台跳过单列，不据此宣称该平台已实测。

### TASK-16 场景模板与浏览器交接

**需求与设计**：[产品 REQ-16](../../01-需求文档/需求.md)；B1/REQ-69、B1/REQ-70、B1/REQ-71、B1/REQ-72；[B1/PLAN-§16](../specs/2026-11-23-existing-system-baseline-design.md#plan-16-场景模板与浏览器交接)。
**已存在文件 / 职责**：[examples/recipes/](../../../examples/recipes/)、[examples/skills/](../../../examples/skills/)、[examples/watch-github-issues.sh](../../../examples/watch-github-issues.sh)、[examples/worker.sh](../../../examples/worker.sh)、[examples/channel.sh](../../../examples/channel.sh)、[examples/plugins/](../../../examples/plugins/)、[skills/agent-autoloop/SKILL.md](../../../skills/agent-autoloop/SKILL.md)。
**输入**：沿用 spec 的当前接口契约和可信配置；不新增相邻模块 API。
**输出 / 实现组**：B1/CODE-P，上述入口文件、脚本及资料；不产生本轮新的功能实现。

- [x] **定位实现**：在上述基准确认入口及职责，沿现有依赖调用；每个 recipe 自带规则、必要脚本、任务 Skill 和启用说明。浏览器入口由可信配置指定，任务与网页内容不能改变目标或允许操作；受阻时主任务 ask 后结束，回答后只继续剩余步骤。
- [x] **定位验证依据**：[tests/recipes.test.ts](../../../tests/recipes.test.ts)、[tests/business-recipes.test.ts](../../../tests/business-recipes.test.ts)、[tests/browser-handoff.test.ts](../../../tests/browser-handoff.test.ts)、[tests/examples.test.ts](../../../tests/examples.test.ts)、[tests/example-plugins.test.ts](../../../tests/example-plugins.test.ts)；核对断言：网站失败只首次唤醒、恢复结案；飞书快照关联原事件；浏览器任务保持原会话，不重复委派。
- [ ] **固定提交后做交付复核**：对本任务检查需求→设计、设计→任务、任务→代码及三个逆向映射；若发现差距先记录，不在本追溯任务内擅自改代码。

**复现命令（按需，不代表本次已跑）**：`node --disable-warning=ExperimentalWarning --test tests/recipes.test.ts tests/business-recipes.test.ts tests/browser-handoff.test.ts tests/examples.test.ts tests/example-plugins.test.ts`。
**通过口径**：对应行为测试无失败；平台跳过单列，不据此宣称该平台已实测。

### TASK-17 日志投影、兼容性与工程验证

**需求与设计**：[非功能需求](../../01-需求文档/需求.md#非功能需求)；B1/REQ-73、B1/REQ-76、B1/REQ-78；[B1/PLAN-§17](../specs/2026-11-23-existing-system-baseline-design.md#plan-17-日志投影兼容性与工程验证)。
**已存在文件 / 职责**：[kernel/store.ts](../../../kernel/store.ts)、[kernel/evolve.ts](../../../kernel/evolve.ts)、[kernel/query.ts](../../../kernel/query.ts)、[kernel/credentials.ts](../../../kernel/credentials.ts)、[sim/](../../../sim/)、[scripts/](../../../scripts/)、[tests/](../../../tests/)、[.github/](../../../.github/)、[package.json](../../../package.json)、[package-lock.json](../../../package-lock.json)、[tsconfig.json](../../../tsconfig.json)、[.gitignore](../../../.gitignore)、[.gitattributes](../../../.gitattributes)、[CONTRIBUTING.md](../../../CONTRIBUTING.md)。
**输入**：当前 View、Input、注入时间与 Environment，以及仓库工程约束。
**输出 / 实现组**：B1/CODE-Q，`commit`、`commitLatest`、`evolve`、`openView`、`rebuild` 的现有职责；不产生本轮新的功能实现。

- [x] **定位实现**：在上述基准确认入口及职责，沿现有依赖调用；commitLatest 在同一写事务内取得最新视图；commit 只追加事实，evolve 独占持久投影。WAL+FULL 逐笔提交；格式变化逐条重建而非业务重放。工程验证沿用日常、模拟、穷举、强杀、变异和跨平台 CI，不增加第二套内核或达标线。
- [x] **定位验证依据**：[tests/kernel.test.ts](../../../tests/kernel.test.ts)、[tests/projection.test.ts](../../../tests/projection.test.ts)、[tests/format.test.ts](../../../tests/format.test.ts)、[tests/purity.test.ts](../../../tests/purity.test.ts)、[tests/ci-scope.test.ts](../../../tests/ci-scope.test.ts)、[tests/simulation.test.ts](../../../tests/simulation.test.ts)、[sim/crash.test.ts](../../../sim/crash.test.ts)、[sim/mutation.test.ts](../../../sim/mutation.test.ts)；核对断言：已确认序号与重建投影一致；投影失败回滚日志；指纹仅受持久格式材料影响；依赖无运行时包。
- [ ] **固定提交后做交付复核**：对本任务检查需求→设计、设计→任务、任务→代码及三个逆向映射；若发现差距先记录，不在本追溯任务内擅自改代码。

**复现命令（按需，不代表本次已跑）**：`npm run test:quick`。
**通过口径**：相应命令退出 0，报告真实测试数与跳过原因；全量模拟和 nightly 仅在约定范围需要时执行。

### TASK-18 文档基线与追溯入口

**需求与设计**：[非功能需求](../../01-需求文档/需求.md#非功能需求)；B1/REQ-75；[B1/PLAN-§18](../specs/2026-11-23-existing-system-baseline-design.md#plan-18-文档基线与追溯入口)。
**已存在文件 / 职责**：[docs/](../../../docs/)、[README.md](../../../README.md)、[README.en.md](../../../README.en.md)、[AGENTS.md](../../../AGENTS.md)、[INSTALL.md](../../../INSTALL.md)、[SECURITY.md](../../../SECURITY.md)、[LICENSE](../../../LICENSE)、[skills/agent-autoloop/SKILL.md](../../../skills/agent-autoloop/SKILL.md)。
**输入**：固定基准需求、代码、测试和原有审阅记录。
**输出 / 实现组**：B1/CODE-R，上述入口文件、脚本及资料；不产生本轮新的功能实现。

- [x] **定位实现**：在上述基准确认入口及职责，沿现有依赖调用；需求、业务、技术、部署、使用和协议各自保持职责。基线 spec 和任务表明确标为事后整理，关联现有需求及固定实现，不冒充历史实施批准；原审阅报告保留当时状态和验证窗口。页面计划单列，未完成设计不伪造为本次交付。
- [x] **定位验证依据**：本 spec/plan、需求及文档总索引；核对断言：每个独立验收条目有来源、设计节、任务和实现证据；本地链接、标题锚点、UTF-8 及围栏有效；不签发未完成的 DELIVER。
- [ ] **固定提交后做交付复核**：对本任务检查需求→设计、设计→任务、任务→代码及三个逆向映射；若发现差距先记录，不在本追溯任务内擅自改代码。

**复现命令（按需，不代表本次已跑）**：`git diff --check`。
**通过口径**：所有映射和引用有效；不出现假定的历史批准、假造的执行记录或自动 DELIVER。


## 支撑文件归属与完整性

基准目录和完整文件路径已见[全仓审阅记录](../../06-交付审计/2026-11-23-文档审阅.md#完整文件清单)。那份清单不含其报告自身；本基准另把报告自身纳入 TASK-18，总计 278 个现存文件。

| 范围 | 归属 |
| --- | --- |
| kernel/ | TASK-1—4、6—9、12—13、17；以逐任务的函数和职责为准 |
| protocol/ | TASK-1—2、4、8、13；Job 为 TASK-4 的执行交接契约 |
| host/ | 配置/服务 TASK-14；状态 TASK-11；规则执行 TASK-2、17；HTTP/插件/分帧 TASK-13；通道 TASK-6；进程辅助 TASK-10 |
| worker/ | 契约 TASK-2、8；适配 TASK-5；运行、恢复、用量及 Windows TASK-10 |
| plugins/ | Agent TASK-5；通道 TASK-6；来源 TASK-13；delivery 全目录 TASK-15 |
| cli/、bin/、install.sh、install.ps1 | 状态/历史 TASK-11；MCP TASK-13；控制 TASK-12；初始化、维护及其余入口 TASK-14 |
| examples/、skills/ | TASK-16；插件/协议样例同时支撑 TASK-13；正式手册也支撑 TASK-18 |
| tests/ | 各任务列出的行为证据；测试辅助、fixture 及未逐项重列的回归测试由 TASK-17 统一支撑，不当成新增业务行为 |
| sim/、scripts/、.github/ | TASK-17；安装冒烟与真机入口同时支持 TASK-14、15 |
| package.json、package-lock.json、tsconfig.json、.gitignore、.gitattributes、CONTRIBUTING.md | TASK-17 的依赖、类型、格式、忽略及贡献/CI 约束 |
| docs/、两份 README、AGENTS.md、INSTALL.md、SECURITY.md、LICENSE | TASK-18 文档基线及现有规则；AGENTS 的边界、贡献约定同时支持 TASK-17 |

这张表覆盖当前文件职责，不能替代正式审计按函数/diff 识别的未映射变更。当前树以外的历史删除和旧方案另列待核对，不宣称其为本计划曾执行的任务。

## 实际验证台账

| 证据 | 范围 | 本轮如何使用 |
| --- | --- | --- |
| 2026-11-23 17:47:03—17:48:06 快速测试 | 97 文件，851 项，830 通过、21 跳过、0 失败 | 引用原审阅报告，不伪称本轮重跑 |
| 同轮类型 / 格式 | 各 1 项，通过 | 仅支持未变化的工程实现，不校验 Markdown 语义 |
| 20 项 Windows + 1 项真实 fish | 原记录明确跳过 | 保留跳过，不补勾实机通过 |
| 模拟、穷举、deep/nightly、安装冒烟 | 前次文档任务没有运行 | 不填执行时间或成功结论 |
| 真实 Agent、平台消息、交付与浏览器 | 未使用真实账号验收 | 不发真实请求来制造证据 |
| 新增基线资料 | 本次编辑后检查 ID、来源、路径、定义、链接和围栏 | 校验完成后在对话给实际结果，正式报告在提交后生成 |

## 本次资料交付顺序

1. 写入两份基线并同步总索引。
2. 自检所有 78 个独立需求 ID、18 个设计节、18 个任务和 18 个实现组；确认使用的路径与函数真实存在。
3. 检查当前文档本地链接、锚点、UTF-8、代码围栏和差异空白；确认源码、测试和依赖清单没有改动。
4. 将文件及提交说明交用户确认；未获提交授权不自动 add/commit。
5. 提交后重新固定审查 SHA，从首次全量范围完成主链、reference 与遗留核对；按实际结果决定是否有资格推进基线。

**交接限制**：不需要选择产品实现的执行方式，因为本任务不实施功能。若后续要开发中控页面或修复真实差距，应另做需求和设计，不复用本文件中的“已存在实现”勾选充当新的任务完成记录。
