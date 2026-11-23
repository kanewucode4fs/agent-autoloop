# 现有系统基线设计（事后整理）

## 文档性质与来源

- **状态**：按用户 2026-11-23 明确授权补齐；基线整理完成后仍需提交和交付一致性复核，不是已通过审计。
- **实现基准**：`71cc1d24f020ef9d9de9f1b395e367e3452dd61a`，分支 `feature/web-console`，阶段 `iteration`。
- **需求来源**：[需求文档](../../01-需求文档/需求.md) 的已实现 REQ-1—REQ-16、六条非功能要求，以及本次补齐追溯材料的用户指令。
- **关联任务表**：[现有系统基线 plan](../plans/2026-11-23-existing-system-baseline.md)。
- 本文是对已有代码和行为测试的**事后设计整理**。不声称这些设计在历史代码之前获批，不伪造原实施顺序、失败测试或历史验收。
- 基线范围是当前已实现系统；272 条历史提交的原授权和 19 个当前已不存在的历史路径，仍须正式审计按历史证据核对，不能仅凭本文自动证明。
- REQ-17 中控页面保持后续目标，不属于本次已实现交付。本文没有决定前端技术栈、界面布局、登录方案或上线方式。

## 目标、方案与边界

把当前系统的需求、设计、任务和实现放进可追溯主链，供后续交付检查逐项核对。用户选择的是“按现有实现整理两份基线”，没有授权重做产品功能。

采用一个总基线 spec 和一份按功能拆分的任务表，保留 kernel、host、worker、protocol、插件的现有划分。与按每个历史提交重建原计划相比，这种方式能准确描述现在的实现，但不补造历史批准；与扩写已有协议相比，它单列设计—任务映射，避免把通信协议当成全部工程计划。

**不做**：新页面、新服务、新依赖、目录重构、数据迁移、外部平台操作、自动提交或签发 DELIVER。现有协议和测试是行为证据，不为让基线“通过”修改预期。

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

## 接口与整体设计

```text
事件 / 时钟 / 回答 / 回执
           ↓
HTTP 鉴权 → decide(view, input, now, env)
           ↓ 本次 facts
commitLatest / commit → evolve → SQLite 投影与索引
           ↓
Job / 卡片 / 本人状态 / SSE
           ↓
外部 worker、通道、CLI、MCP 客户端
```

| 契约 | 当前入口 | 结果 / 归属 |
| --- | --- | --- |
| 确定性判断 | `decide(view: View, input: Input, now: number, env: Environment)` | `{ facts: Fact[]; value: any }`，不直接写持久投影 |
| 原子提交 | `commitLatest(db, format, produce)`、`commit(db, view, produce)` | 返回新 View，判断和提交保持同一事务 |
| 投影演进 | `evolve(view, facts, db, reset)` | 唯一业务投影持久写入点 |
| 接单 | `createWorker(options: WorkerOptions)` | 返回 tick/run/entries/tidy；默认 capacity=2 |
| Agent | `Adapter.command(turn: Turn): string[]`；`read(stdout, code, stderr): Reading` | CLI 参数、最后消息、会话、用量、started/finished |
| 中台客户端 | `hubClient(url: string, key: string)` | 类型 Hub；所有外部角色用同一组 HTTP |
| Job | [protocol/job.ts](../../../protocol/job.ts) | run/token/attempt、可信 rule/agent、session/askFirst、context 资料 |
| 本人查询 | `stateOf`、`why`、`logAfter` | 快照 seq 与 SSE after 相接，查询不作业务决定 |

详细字段、鉴权和错误码沿用[协议](../../protocol.md)，状态归属沿用[状态机](../../state-machines.md)，物理存储沿用[技术架构](../../03-技术架构/架构详解.md)。

## 独立验收条目与主链 ID

原需求文档一段可能包含多个验收行为，以下拆成 78 个审查条目，不替换其原编号。使用 B1 前缀防止与后续事项混淆；同一条目可有补充关联，表中列主关联。

| 审查需求 ID | 原文来源 | 独立验收要求 | 设计节 → 任务 → 实现 |
| --- | --- | --- | --- |
| B1/REQ-1 | REQ-1 | 缺少必填字段或超出协议边界的事件被拒收，且不留下半条业务记录。 | B1/PLAN-§1 → B1/TASK-1 → B1/CODE-A |
| B1/REQ-2 | REQ-1 | 同来源同 id 重发只记录并路由一次，重复提交获得稳定确认。 | B1/PLAN-§1 → B1/TASK-1 → B1/CODE-A |
| B1/REQ-3 | REQ-1 | 来源身份不能冒充其他来源，只能查自己发出的事件。 | B1/PLAN-§1 → B1/TASK-1 → B1/CODE-A |
| B1/REQ-4 | REQ-2 | 规则引用不存在的来源、Agent 或通道时报可定位的错误。 | B1/PLAN-§2 → B1/TASK-2 → B1/CODE-B |
| B1/REQ-5 | REQ-2 | 外部 summary/detail 等正文放资料文件，不拼入可信指令段。 | B1/PLAN-§2 → B1/TASK-2 → B1/CODE-B |
| B1/REQ-6 | REQ-2 | 外部资料不能改写 Agent、cwd、access、插件路径或交付仓库。 | B1/PLAN-§2 → B1/TASK-2 → B1/CODE-B |
| B1/REQ-7 | REQ-2 | 新规则无效、缺失或暂不可读时，已有合法规则继续生效并报告问题。 | B1/PLAN-§2 → B1/TASK-2 → B1/CODE-B |
| B1/REQ-8 | REQ-3 | cron 只在到期时产生相应事件或轮次。 | B1/PLAN-§3 → B1/TASK-3 → B1/CODE-C |
| B1/REQ-9 | REQ-3 | 同一个计划时刻不会重复触发。 | B1/PLAN-§3 → B1/TASK-3 → B1/CODE-C |
| B1/REQ-10 | REQ-3 | 睡眠错过多个计划时刻时只补最近一次，不批量回放所有错过时刻。 | B1/PLAN-§3 → B1/TASK-3 → B1/CODE-C |
| B1/REQ-11 | REQ-3 | 时区、夏令时重复和不存在的本地时刻按协议解释。 | B1/PLAN-§3 → B1/TASK-3 → B1/CODE-C |
| B1/REQ-12 | REQ-4 | 未设 parallel 时整条规则归为一个事项。 | B1/PLAN-§4 → B1/TASK-4 → B1/CODE-D |
| B1/REQ-13 | REQ-4 | 设置 parallel 后按事件 subject 分事项。 | B1/PLAN-§4 → B1/TASK-4 → B1/CODE-D |
| B1/REQ-14 | REQ-4 | 同一事项不能同时领取两轮。 | B1/PLAN-§4 → B1/TASK-4 → B1/CODE-D |
| B1/REQ-15 | REQ-4 | 组 parallel、规则 maxActive、Agent maxActive 和 worker capacity 的上限同时生效。 | B1/PLAN-§4 → B1/TASK-4 → B1/CODE-D |
| B1/REQ-16 | REQ-5 | 工作目录和各命令参数按可信配置原样传递，不被重新解释成 shell 语法。 | B1/PLAN-§5 → B1/TASK-5 → B1/CODE-E |
| B1/REQ-17 | REQ-5 | read/edit/full 只按对应适配器的约定映射，不无声扩大权限。 | B1/PLAN-§5 → B1/TASK-5 → B1/CODE-E |
| B1/REQ-18 | REQ-5 | 非零退出或没有可用最终消息时，给出实际失败结果。 | B1/PLAN-§5 → B1/TASK-5 → B1/CODE-E |
| B1/REQ-19 | REQ-5 | 思考、工具输出、历史回放和未结束后台工作不冒充本轮最终回答。 | B1/PLAN-§5 → B1/TASK-5 → B1/CODE-E |
| B1/REQ-20 | REQ-6 | 同一事项同 key 的未结束提问不重复创建。 | B1/PLAN-§6 → B1/TASK-6 → B1/CODE-F |
| B1/REQ-21 | REQ-6 | 通道转达回答时核对允许的回答人。 | B1/PLAN-§6 → B1/TASK-6 → B1/CODE-F |
| B1/REQ-22 | REQ-6 | 通道回答引用与已送达原消息 ref 匹配。 | B1/PLAN-§6 → B1/TASK-6 → B1/CODE-F |
| B1/REQ-23 | REQ-6 | 重复回答或回答与结案竞争时只接受一次有效变更。 | B1/PLAN-§6 → B1/TASK-6 → B1/CODE-F |
| B1/REQ-24 | REQ-7 | 本人回答叫醒原提问事项，而不是当前规则重新计算出的其他 lane。 | B1/PLAN-§7 → B1/TASK-7 → B1/CODE-G |
| B1/REQ-25 | REQ-7 | 允许续接时使用原 Agent 的原会话和已有资料。 | B1/PLAN-§7 → B1/TASK-7 → B1/CODE-G |
| B1/REQ-26 | REQ-7 | 换 Agent 或会话可能仍被使用时，不误用旧会话。 | B1/PLAN-§7 → B1/TASK-7 → B1/CODE-G |
| B1/REQ-27 | REQ-7 | 只有后续轮次成功处理回答，卡片才表示决定已执行。 | B1/PLAN-§7 → B1/TASK-7 → B1/CODE-G |
| B1/REQ-28 | REQ-8 | 成功回执的结果和允许的后续动作在同一事务生效。 | B1/PLAN-§8 → B1/TASK-8 → B1/CODE-H |
| B1/REQ-29 | REQ-8 | 同 token 同内容重交返回同一确认，不同内容返回冲突。 | B1/PLAN-§8 → B1/TASK-8 → B1/CODE-H |
| B1/REQ-30 | REQ-8 | 回执格式错误仅请求重写最终消息，不重新执行业务。 | B1/PLAN-§8 → B1/TASK-8 → B1/CODE-H |
| B1/REQ-31 | REQ-8 | 无法重写或仍无有效回执时，保留原业务结局并报告，不把已做业务自动重做。 | B1/PLAN-§8 → B1/TASK-8 → B1/CODE-H |
| B1/REQ-32 | REQ-9 | 普通失败从 5 分钟开始退避，最大重试间隔 1 小时。 | B1/PLAN-§9 → B1/TASK-9 → B1/CODE-I |
| B1/REQ-33 | REQ-9 | 连续第 3 次失败通知本人。 | B1/PLAN-§9 → B1/TASK-9 → B1/CODE-I |
| B1/REQ-34 | REQ-9 | 连续第 10 次失败结束该轮，不再自动重派。 | B1/PLAN-§9 → B1/TASK-9 → B1/CODE-I |
| B1/REQ-35 | REQ-9 | 失败期间同事项新事件合并进重试轮。 | B1/PLAN-§9 → B1/TASK-9 → B1/CODE-I |
| B1/REQ-36 | REQ-9 | 额度限制在恢复前阻止给该 Agent 继续派活。 | B1/PLAN-§9 → B1/TASK-9 → B1/CODE-I |
| B1/REQ-37 | REQ-9 | 本人停止的一轮交 failed 也归取消，不重新排队。 | B1/PLAN-§9 → B1/TASK-9 → B1/CODE-I |
| B1/REQ-38 | REQ-10 | worker 重启后继续观察原进程，不重新启动已经在跑的业务。 | B1/PLAN-§10 → B1/TASK-10 → B1/CODE-J |
| B1/REQ-39 | REQ-10 | 已经跑完但未交出的结果保存在本地，并在恢复后继续提交。 | B1/PLAN-§10 → B1/TASK-10 → B1/CODE-J |
| B1/REQ-40 | REQ-10 | 进程查询不可用或权限不足时，不能认定进程已经结束。 | B1/PLAN-§10 → B1/TASK-10 → B1/CODE-J |
| B1/REQ-41 | REQ-10 | 停止确认在整个子进程树退出之后发生。 | B1/PLAN-§10 → B1/TASK-10 → B1/CODE-J |
| B1/REQ-42 | REQ-10 | 已接回会话后中断时，不换新会话重做已开始工作。 | B1/PLAN-§10 → B1/TASK-10 → B1/CODE-J |
| B1/REQ-43 | REQ-11 | status --json 输出当前状态快照及相应日志序号。 | B1/PLAN-§11 → B1/TASK-11 → B1/CODE-K |
| B1/REQ-44 | REQ-11 | 可按规则或 before 游标读取更早轮次，不混入仍在排队或领取的规则历史。 | B1/PLAN-§11 → B1/TASK-11 → B1/CODE-K |
| B1/REQ-45 | REQ-11 | why 将事件、规则版本、执行、回执和回答关联成时间线。 | B1/PLAN-§11 → B1/TASK-11 → B1/CODE-K |
| B1/REQ-46 | REQ-11 | 用量按完成时间及规则时区汇总今天和最近七天。 | B1/PLAN-§11 → B1/TASK-11 → B1/CODE-K |
| B1/REQ-47 | REQ-11 | 未上报用量或缺可信基线明确保留缺失，不伪造为零。 | B1/PLAN-§11 → B1/TASK-11 → B1/CODE-K |
| B1/REQ-48 | REQ-12 | 暂停中不为该规则接新来源事件，也不派排队任务。 | B1/PLAN-§12 → B1/TASK-12 → B1/CODE-L |
| B1/REQ-49 | REQ-12 | 已有队列、回答和到期复查保留，已领取任务不受暂停直接终止。 | B1/PLAN-§12 → B1/TASK-12 → B1/CODE-L |
| B1/REQ-50 | REQ-12 | 恢复不自动补派暂停跳过的来源事件或定时时刻。 | B1/PLAN-§12 → B1/TASK-12 → B1/CODE-L |
| B1/REQ-51 | REQ-12 | replay 按 since 补跳过事件，同一事件对该规则只补一次。 | B1/PLAN-§12 → B1/TASK-12 → B1/CODE-L |
| B1/REQ-52 | REQ-12 | 暂停时不能用 replay 或 run 绕过暂停。 | B1/PLAN-§12 → B1/TASK-12 → B1/CODE-L |
| B1/REQ-53 | REQ-13 | source/worker/channel/owner 各类密钥不能跨角色访问。 | B1/PLAN-§13 → B1/TASK-13 → B1/CODE-M |
| B1/REQ-54 | REQ-13 | 轮次 token 只对这一轮的这次执行有效。 | B1/PLAN-§13 → B1/TASK-13 → B1/CODE-M |
| B1/REQ-55 | REQ-13 | 插件安装只克隆，不执行其安装脚本或额外依赖安装。 | B1/PLAN-§13 → B1/TASK-13 → B1/CODE-M |
| B1/REQ-56 | REQ-13 | 数据根 plugins 的同名插件优先于引擎内置插件。 | B1/PLAN-§13 → B1/TASK-13 → B1/CODE-M |
| B1/REQ-57 | REQ-13 | MCP 仅展示本机持有身份允许的固定操作，不开放任意地址执行。 | B1/PLAN-§13 → B1/TASK-13 → B1/CODE-M |
| B1/REQ-58 | REQ-14 | init 拒绝覆盖已有配置，并校验指定程序可用。 | B1/PLAN-§14 → B1/TASK-14 → B1/CODE-N |
| B1/REQ-59 | REQ-14 | join 不覆盖已经配置的中台或接单电脑。 | B1/PLAN-§14 → B1/TASK-14 → B1/CODE-N |
| B1/REQ-60 | REQ-14 | macOS、Linux、Windows 使用各自当前用户的后台机制。 | B1/PLAN-§14 → B1/TASK-14 → B1/CODE-N |
| B1/REQ-61 | REQ-14 | backup 可在中台运行时取得一致数据库及存在的配套秘密文件。 | B1/PLAN-§14 → B1/TASK-14 → B1/CODE-N |
| B1/REQ-62 | REQ-14 | restore 在中台仍占用数据库时拒绝，替换须确认并保留原文件。 | B1/PLAN-§14 → B1/TASK-14 → B1/CODE-N |
| B1/REQ-63 | REQ-14 | uninstall 移除自身程序、链接和服务，保留规则、密钥和历史。 | B1/PLAN-§14 → B1/TASK-14 → B1/CODE-N |
| B1/REQ-64 | REQ-15 | 交付设置和审查点只读取固定 main 提交。 | B1/PLAN-§15 → B1/TASK-15 → B1/CODE-O |
| B1/REQ-65 | REQ-15 | 审查只读取裸 Git 文字快照，不检出或运行 PR 的代码、测试、配置和 hooks。 | B1/PLAN-§15 → B1/TASK-15 → B1/CODE-O |
| B1/REQ-66 | REQ-15 | 只接受当前 head 的最新 pull_request 必需检查，草稿及 skipped 不放行。 | B1/PLAN-§15 → B1/TASK-15 → B1/CODE-O |
| B1/REQ-67 | REQ-15 | 合并前重新核对 head、main、base、标签和自动合并状态。 | B1/PLAN-§15 → B1/TASK-15 → B1/CODE-O |
| B1/REQ-68 | REQ-15 | 实际上线需要覆盖目标提交的 Ship 成功和健康版本一起确认。 | B1/PLAN-§15 → B1/TASK-15 → B1/CODE-O |
| B1/REQ-69 | REQ-16 | 场景规则通过现有 parseRules/check 校验。 | B1/PLAN-§16 → B1/TASK-16 → B1/CODE-P |
| B1/REQ-70 | REQ-16 | 样例脚本使用本地假 HTTP 和假平台 CLI 验证，不向真实账号发消息。 | B1/PLAN-§16 → B1/TASK-16 → B1/CODE-P |
| B1/REQ-71 | REQ-16 | 启用前替换目录、身份和业务地址，不将真实凭据写入仓库。 | B1/PLAN-§16 → B1/TASK-16 → B1/CODE-P |
| B1/REQ-72 | REQ-16 | 样例验证不冒充真实账户、浏览器或部署平台的端到端验收。 | B1/PLAN-§16 → B1/TASK-16 → B1/CODE-P |
| B1/REQ-73 | 非功能：数据可靠性 | 事实日志为业务权威；日志与投影同事务，失败回滚，重建不重新执行业务。 | B1/PLAN-§17 → B1/TASK-17 → B1/CODE-Q |
| B1/REQ-74 | 非功能：权限边界 | 凭据按角色分开，规则是指令；卡片页面转义外来文字，打开链接不产生回答。 | B1/PLAN-§13 → B1/TASK-13 → B1/CODE-M |
| B1/REQ-75 | 非功能：可追溯 | 规则版本、事件、回执、回答可关联；状态快照可从其日志序号继续订阅。 | B1/PLAN-§11 → B1/TASK-11 → B1/CODE-K |
| B1/REQ-76 | 非功能：兼容性 | 既有 /v1、规则、插件、命令和历史数据按协议兼容约定延续。 | B1/PLAN-§17 → B1/TASK-17 → B1/CODE-Q |
| B1/REQ-77 | 非功能：依赖与部署 | 运行要求 Node >=24，不新增运行时 npm 包；使用现有原生部署方式。 | B1/PLAN-§14 → B1/TASK-14 → B1/CODE-N |
| B1/REQ-78 | 非功能：性能 | 性能只记实测；空闲和千连接口径分开，持久写入逐笔提交，不设达标线。 | B1/PLAN-§17 → B1/TASK-17 → B1/CODE-Q |

非功能“可追溯”还为 §18 的基线资料提供支持；§18 和 TASK-18 是本次用户授权的资料补齐，不新增产品行为。REQ-17 页面以 `B1/FUTURE-1` 记录范围，不分配假实现 ID 或完成任务。

## 功能设计

### PLAN-§1 事件接入与来源结果

**关联**：B1/REQ-1、B1/REQ-2、B1/REQ-3 → B1/TASK-1 → B1/CODE-A。

来源身份来自请求凭据，id 为来源内稳定编号。ingest 校验后匹配可信规则；未匹配、暂停跳过和超过 hops 的事件仍按协议留痕。查询结果须防止泄露混入其他来源的资料。

**代码落点**：[kernel/decide.ts](../../../kernel/decide.ts)、[kernel/query.ts](../../../kernel/query.ts)、[protocol/validate.ts](../../../protocol/validate.ts)。

**函数 / 模块入口**：`ingest`、`recordEvent`、`eventOutcome`、`parseEvent`。
**行为证据**：[tests/events.test.ts](../../../tests/events.test.ts)、[tests/event-outcome.test.ts](../../../tests/event-outcome.test.ts)、[tests/hops.test.ts](../../../tests/hops.test.ts)。
**核对要点**：重复事件只有一个 event/派发；另一来源的事件返回 404；hops 超过 8 留记录、不继续派发。

### PLAN-§2 可信规则与指令资料分离

**关联**：B1/REQ-4、B1/REQ-5、B1/REQ-6、B1/REQ-7 → B1/TASK-2 → B1/CODE-B。

规则校验与类型/匹配分层；规则和 Skill 由本人维护。执行提示词只引用资料文件，业务文本不获得路由或权限。热加载错误保留旧配置；首次没有有效规则则不启动。

**代码落点**：[protocol/rules.ts](../../../protocol/rules.ts)、[kernel/rules.ts](../../../kernel/rules.ts)、[worker/contract.ts](../../../worker/contract.ts)、[host/runtime.ts](../../../host/runtime.ts)。

**函数 / 模块入口**：`parseRules`、`promptFor`、`reloadRules`。
**行为证据**：[tests/rules.test.ts](../../../tests/rules.test.ts)、[tests/contract.test.ts](../../../tests/contract.test.ts)、[tests/rules-preview.test.ts](../../../tests/rules-preview.test.ts)。
**核对要点**：坏引用拒收；prompt 不含事件正文；explain/dry-run 不写中台、不泄露私有任务说明。

### PLAN-§3 定时、时区与到期

**关联**：B1/REQ-8、B1/REQ-9、B1/REQ-10、B1/REQ-11 → B1/TASK-3 → B1/CODE-C。

host 注入时间；cron、复查、退避、租约及离线判断共用 due 索引和外壳单一定时器。规则或时区变化重新计算计划。心跳记录受一分钟节流，不在每次轮询追加事实。

**代码落点**：[kernel/cron.ts](../../../kernel/cron.ts)、[kernel/due.ts](../../../kernel/due.ts)、[kernel/decide.ts](../../../kernel/decide.ts)。

**函数 / 模块入口**：`parseCron`、`nextRun`、`latestBefore`、`deadlines`、`decide`。
**行为证据**：[tests/cron.test.ts](../../../tests/cron.test.ts)、[tests/schedule.test.ts](../../../tests/schedule.test.ts)、[tests/heartbeat.test.ts](../../../tests/heartbeat.test.ts)。
**核对要点**：固定注入时间决定触发；时钟恢复只补最近时刻；夏令时重复取第一次、不存在的时刻跳过。

### PLAN-§4 事项、合并和并发领取

**关联**：B1/REQ-12、B1/REQ-13、B1/REQ-14、B1/REQ-15 → B1/TASK-4 → B1/CODE-D。

事项以 rule+lane 表示，没有独立任务状态表。同事项排队事件合并；在跑期间的新事件排后续轮。领取时采用当前规则，领取后保持执行归属；重复 claim 仍拿同一工作但更新能力 token。

**代码落点**：[kernel/decide.ts](../../../kernel/decide.ts)、[kernel/query.ts](../../../kernel/query.ts)、[worker/worker.ts](../../../worker/worker.ts)、[protocol/job.ts](../../../protocol/job.ts)。

**函数 / 模块入口**：`enqueue`、`claimNext`、`assignQueued`、`jobOf`、`createWorker`。
**行为证据**：[tests/runs.test.ts](../../../tests/runs.test.ts)、[tests/queue-costs.test.ts](../../../tests/queue-costs.test.ts)、[tests/worker.test.ts](../../../tests/worker.test.ts)。
**核对要点**：同 lane 不并发；同 claim 不重复派活；多层限额均受测试约束；worker 默认 capacity=2。

### PLAN-§5 Agent 插件与进程结果读取

**关联**：B1/REQ-16、B1/REQ-17、B1/REQ-18、B1/REQ-19 → B1/TASK-5 → B1/CODE-E。

适配器以 Adapter.command/read/link 描述专用差异；通用 command 只保证进程契约，不替任意程序限制权限。ACP 读协议 v1，会话历史不能变成本轮消息。CLI 自身登录和工具由执行环境提供。

**代码落点**：[worker/agents.ts](../../../worker/agents.ts)、[worker/worker.ts](../../../worker/worker.ts)、[plugins/command/agent.ts](../../../plugins/command/agent.ts)、[plugins/claude/agent.ts](../../../plugins/claude/agent.ts)、[plugins/codex/agent.ts](../../../plugins/codex/agent.ts)、[plugins/acp/agent.ts](../../../plugins/acp/agent.ts)、[plugins/acp/bridge.ts](../../../plugins/acp/bridge.ts)、[plugins/dsh/agent.ts](../../../plugins/dsh/agent.ts)、[plugins/zcode/agent.ts](../../../plugins/zcode/agent.ts)。

**函数 / 模块入口**：`adapterFor`、`findProgram`、`createWorker`、`launch`、`conclude`。
**行为证据**：[tests/adapters.test.ts](../../../tests/adapters.test.ts)、[tests/acp.test.ts](../../../tests/acp.test.ts)、[tests/dsh.test.ts](../../../tests/dsh.test.ts)、[tests/zcode.test.ts](../../../tests/zcode.test.ts)、[tests/worker-path.test.ts](../../../tests/worker-path.test.ts)。
**核对要点**：参数含空格/特殊字符仍完整；未结束输出不算完成；恢复会话与权限模式按适配器逐项验证。

### PLAN-§6 卡片创建、投递和本人回答

**关联**：B1/REQ-20、B1/REQ-21、B1/REQ-22、B1/REQ-23 → B1/TASK-6 → B1/CODE-F。

内容创建后不变，版本只表达状态变更。提问 open→decided→done，直接结案 open→closed；FYI 创建即 closed。通道报告版本和 ref；卡片表单转义外来内容、不执行脚本，GET 不改变决定。

**代码落点**：[kernel/decide.ts](../../../kernel/decide.ts)、[kernel/card-page.ts](../../../kernel/card-page.ts)、[host/channels.ts](../../../host/channels.ts)、[plugins/desktop/channel.ts](../../../plugins/desktop/channel.ts)、[plugins/feishu/channel.ts](../../../plugins/feishu/channel.ts)、[plugins/webhook/channel.ts](../../../plugins/webhook/channel.ts)、[plugins/webhook/templates.ts](../../../plugins/webhook/templates.ts)。

**函数 / 模块入口**：`createCard`、`nextCard`、`answer`、`closeCard`。
**行为证据**：[tests/cards.test.ts](../../../tests/cards.test.ts)、[tests/channels.test.ts](../../../tests/channels.test.ts)、[tests/desktop.test.ts](../../../tests/desktop.test.ts)、[tests/webhook-templates.test.ts](../../../tests/webhook-templates.test.ts)。
**核对要点**：重复问题保持原卡；越权回答被拒；链接从创建起 7 天有效；假通知记录一次，不发真消息。

### PLAN-§7 回答续办与会话归属

**关联**：B1/REQ-24、B1/REQ-25、B1/REQ-26、B1/REQ-27 → B1/TASK-7 → B1/CODE-G。

提问结束本轮，不阻塞 Agent 等人。回答和复查携带原 lane、Agent 及会话归属；会话冲突时保留资料并开新会话。elsewhere 续办使用配置的任务上下文，不接外部定时任务自己的会话。

**代码落点**：[kernel/decide.ts](../../../kernel/decide.ts)、[kernel/query.ts](../../../kernel/query.ts)、[worker/worker.ts](../../../worker/worker.ts)。

**函数 / 模块入口**：`answer`、`settleSession`、`applyReceipt`、`askedAbout`、`jobOf`。
**行为证据**：[tests/resume-thread.test.ts](../../../tests/resume-thread.test.ts)、[tests/stops-and-threads.test.ts](../../../tests/stops-and-threads.test.ts)、[tests/loop.test.ts](../../../tests/loop.test.ts)、[tests/elsewhere-ask.test.ts](../../../tests/elsewhere-ask.test.ts)。
**核对要点**：绿色回答回到原事项，卡片从 decided 变 done；跨 Agent 复查不能沿用旧 Agent 会话。

### PLAN-§8 回执、重放与后续动作

**关联**：B1/REQ-28、B1/REQ-29、B1/REQ-30、B1/REQ-31 → B1/TASK-8 → B1/CODE-H。

普通 Agent 输出第一行结论及可选 agent-autoloop 代码块。ask/notice/resolve/later/events 由中台校验范围；failed 不生效提问、结案、复查和新事件。支持会话的修复最多两次，emits 与 hops 不能由回执自定越权。

**代码落点**：[protocol/validate.ts](../../../protocol/validate.ts)、[worker/contract.ts](../../../worker/contract.ts)、[worker/worker.ts](../../../worker/worker.ts)、[kernel/decide.ts](../../../kernel/decide.ts)、[kernel/store.ts](../../../kernel/store.ts)、[kernel/evolve.ts](../../../kernel/evolve.ts)。

**函数 / 模块入口**：`parseResult`、`readMessage`、`resultFrom`、`conclude`、`applyReceipt`、`finish`、`commit`、`evolve`。
**行为证据**：[tests/contract.test.ts](../../../tests/contract.test.ts)、[tests/worker.test.ts](../../../tests/worker.test.ts)、[tests/kernel.test.ts](../../../tests/kernel.test.ts)、[tests/stream.test.ts](../../../tests/stream.test.ts)。
**核对要点**：事务失败全部回滚；回执重放确认相同；修复最多 2 次；超过 8 hops 只记不派。

### PLAN-§9 失败重派、额度与停止

**关联**：B1/REQ-32、B1/REQ-33、B1/REQ-34、B1/REQ-35、B1/REQ-36、B1/REQ-37 → B1/TASK-9 → B1/CODE-I。

重试属于中台；worker 不建立另一套业务重试。limitedUntil 必须是有效窗口内的带时区时间。需要本人恢复登录、连接或验证应 done+ask；unknown 不等于失败，迟到结果仍按能力核验接收。

**代码落点**：[kernel/decide.ts](../../../kernel/decide.ts)、[kernel/due.ts](../../../kernel/due.ts)、[worker/worker.ts](../../../worker/worker.ts)。

**函数 / 模块入口**：`retryLater`、`limit`、`lift`、`sweep`、`stopRun`、`finish`。
**行为证据**：[tests/runs.test.ts](../../../tests/runs.test.ts)、[tests/usage-limit.test.ts](../../../tests/usage-limit.test.ts)、[tests/stops-and-threads.test.ts](../../../tests/stops-and-threads.test.ts)、[tests/heartbeat.test.ts](../../../tests/heartbeat.test.ts)。
**核对要点**：失败轮次 attempts/notBefore 正确；额度恢复后继续；停止中失败不重派。

### PLAN-§10 本地执行记录、恢复和清理

**关联**：B1/REQ-38、B1/REQ-39、B1/REQ-40、B1/REQ-41、B1/REQ-42 → B1/TASK-10 → B1/CODE-J。

entry 和结果先持久化，再启动或提交。Unix 跟踪带标记进程组，Windows 通过监督进程和 Job Object 管理后代。默认超时 180 分钟，最终消息后留 30 秒退出，停止宽限 10 秒，完成输出保留 14 天。

**代码落点**：[worker/worker.ts](../../../worker/worker.ts)、[worker/run.ts](../../../worker/run.ts)、[worker/windows.ps1](../../../worker/windows.ps1)、[worker/usage.ts](../../../worker/usage.ts)、[host/command.ts](../../../host/command.ts)、[host/node-path.ts](../../../host/node-path.ts)。

**函数 / 模块入口**：`createWorker`、`recoverEntry`、`launch`、`endWith`、`settle`、`tidy`。
**行为证据**：[tests/worker.test.ts](../../../tests/worker.test.ts)、[tests/worker-lookup.test.ts](../../../tests/worker-lookup.test.ts)、[tests/worker-active.test.ts](../../../tests/worker-active.test.ts)、[tests/worker-tidy.test.ts](../../../tests/worker-tidy.test.ts)、[tests/session-usage.test.ts](../../../tests/session-usage.test.ts)、[tests/windows.test.ts](../../../tests/windows.test.ts)。
**核对要点**：查询失败保留运行状态；恢复不重复业务；缺可信用量基线不冒报累计或零；有 entry 的目录不被清理。

### PLAN-§11 本人状态、历史、事实流和用量

**关联**：B1/REQ-43、B1/REQ-44、B1/REQ-45、B1/REQ-46、B1/REQ-47、B1/REQ-75 → B1/TASK-11 → B1/CODE-K。

host 查询只读视图，不在查询中作业务决定。本人先取 state.seq，再从 stream after 续读；断线可用 Last-Event-ID。快照默认最近 60 轮，CLI runs 每页 20 条，统计不随历史分页截断。

**代码落点**：[host/state.ts](../../../host/state.ts)、[host/runtime.ts](../../../host/runtime.ts)、[cli/status.ts](../../../cli/status.ts)、[cli/runs.ts](../../../cli/runs.ts)、[cli/why.ts](../../../cli/why.ts)、[cli/spend.ts](../../../cli/spend.ts)、[protocol/client.ts](../../../protocol/client.ts)、[kernel/query.ts](../../../kernel/query.ts)。

**函数 / 模块入口**：`stateOf`、`ruleHistory`、`spendingOf`、`why`、`logAfter`、`safeFact`。
**行为证据**：[tests/status-json.test.ts](../../../tests/status-json.test.ts)、[tests/status-view.test.ts](../../../tests/status-view.test.ts)、[tests/status-cost.test.ts](../../../tests/status-cost.test.ts)、[tests/cli-runs.test.ts](../../../tests/cli-runs.test.ts)、[tests/rule-history.test.ts](../../../tests/rule-history.test.ts)、[tests/why.test.ts](../../../tests/why.test.ts)、[tests/stream.test.ts](../../../tests/stream.test.ts)。
**核对要点**：快照与 SSE 序号衔接；历史分页完整；流水不泄露密钥哈希；管道无颜色污染。

### PLAN-§12 暂停、恢复、补跑和手动触发

**关联**：B1/REQ-48、B1/REQ-49、B1/REQ-50、B1/REQ-51、B1/REQ-52 → B1/TASK-12 → B1/CODE-L。

pauseRule/resumeRule/replayRule/runRule 是 decide 的控制输入，在日志内表达，不改 rules.json。恢复应答携带原暂停和跳过概况；until 到期自动恢复。stop 仍是轮次级操作。

**代码落点**：[kernel/decide.ts](../../../kernel/decide.ts)、[kernel/query.ts](../../../kernel/query.ts)、[host/state.ts](../../../host/state.ts)、[cli/main.ts](../../../cli/main.ts)。

**函数 / 模块入口**：`decide`、`pauseOf`、`skippedOf`。
**行为证据**：[tests/pause.test.ts](../../../tests/pause.test.ts)、[tests/pause-http.test.ts](../../../tests/pause-http.test.ts)。
**核对要点**：控制操作只接受 owner；暂停 run/replay 返回 409；恢复给实际 since 以供本人决定是否补跑。

### PLAN-§13 协议、身份、MCP 与统一插件入口

**关联**：B1/REQ-53、B1/REQ-54、B1/REQ-55、B1/REQ-56、B1/REQ-57、B1/REQ-74 → B1/TASK-13 → B1/CODE-M。

HTTP 路由在 kernel，Node 转换和连接在 host。健康接口仅公开 ok/commit。MCP 使用换行 JSON-RPC 和 2025-11-25 握手；同一协议被 CLI、worker 和通道使用。插件只能以合法名字发现，来源验签秘密在外部插件端。

**代码落点**：[kernel/http.ts](../../../kernel/http.ts)、[kernel/credentials.ts](../../../kernel/credentials.ts)、[kernel/errors.ts](../../../kernel/errors.ts)、[protocol/client.ts](../../../protocol/client.ts)、[protocol/schema.ts](../../../protocol/schema.ts)、[protocol/schema.json](../../../protocol/schema.json)、[protocol/conformance.ts](../../../protocol/conformance.ts)、[host/http.ts](../../../host/http.ts)、[host/server.ts](../../../host/server.ts)、[host/plugins.ts](../../../host/plugins.ts)、[host/lines.ts](../../../host/lines.ts)、[cli/mcp.ts](../../../cli/mcp.ts)、[plugins/github/source.ts](../../../plugins/github/source.ts)、[plugins/gitlab/source.ts](../../../plugins/gitlab/source.ts)。

**函数 / 模块入口**：`createHandler`、`whose`、`credential`、`hubClient`、`checkSchema`、`conformance`、`serve`、`makeHandler`、`loadPlugin`。
**行为证据**：[tests/http.test.ts](../../../tests/http.test.ts)、[tests/conformance.test.ts](../../../tests/conformance.test.ts)、[tests/schema.test.ts](../../../tests/schema.test.ts)、[tests/key-remove.test.ts](../../../tests/key-remove.test.ts)、[tests/mcp.test.ts](../../../tests/mcp.test.ts)、[tests/plugins.test.ts](../../../tests/plugins.test.ts)、[tests/github.test.ts](../../../tests/github.test.ts)、[tests/gitlab.test.ts](../../../tests/gitlab.test.ts)、[tests/lines.test.ts](../../../tests/lines.test.ts)。
**核对要点**：错误种类密钥被拒；删除密钥使等待请求失效；来源 HMAC/token 不正确时拒收；Schema 与运行边界一致。

### PLAN-§14 安装、常驻、多机、备份及维护

**关联**：B1/REQ-58、B1/REQ-59、B1/REQ-60、B1/REQ-61、B1/REQ-62、B1/REQ-63、B1/REQ-77 → B1/TASK-14 → B1/CODE-N。

默认数据根 ~/.agent-autoloop，默认回环监听 3350；Unix 权限和 Windows ACL 各自限制数据。默认公共源按正式 tag 安装更新，自定义 origin 按其分支快进。主密钥和数据库必须一起备份；没有容器部署或数据库迁移框架。

**代码落点**：[install.sh](../../../install.sh)、[install.ps1](../../../install.ps1)、[bin/agent-autoloop.ts](../../../bin/agent-autoloop.ts)、[bin/agent-autoloop.cmd](../../../bin/agent-autoloop.cmd)、[cli/main.ts](../../../cli/main.ts)、[cli/update.ts](../../../cli/update.ts)、[cli/backup.ts](../../../cli/backup.ts)、[cli/uninstall.ts](../../../cli/uninstall.ts)、[cli/try.ts](../../../cli/try.ts)、[cli/explain.ts](../../../cli/explain.ts)、[host/up.ts](../../../host/up.ts)、[host/config.ts](../../../host/config.ts)、[host/home.ts](../../../host/home.ts)、[host/files.ts](../../../host/files.ts)、[host/service.ts](../../../host/service.ts)、[host/background.ts](../../../host/background.ts)、[host/background.js](../../../host/background.js)、[host/version.ts](../../../host/version.ts)。

**函数 / 模块入口**：`readConfig`、`writeConfig`、`up`、`installService`、`removeService`、`backup`、`restore`、`updateCode`。
**行为证据**：[tests/install.test.ts](../../../tests/install.test.ts)、[tests/init-command.test.ts](../../../tests/init-command.test.ts)、[tests/cli-flags.test.ts](../../../tests/cli-flags.test.ts)、[tests/join.test.ts](../../../tests/join.test.ts)、[tests/backup.test.ts](../../../tests/backup.test.ts)、[tests/uninstall.test.ts](../../../tests/uninstall.test.ts)、[tests/releases.test.ts](../../../tests/releases.test.ts)、[tests/service.test.ts](../../../tests/service.test.ts)、[tests/loop.test.ts](../../../tests/loop.test.ts)。
**核对要点**：配置文件私有；help 不产生安装副作用；备份恢复包含 WAL 行为；Windows 原生与登录服务需对应平台另验。

### PLAN-§15 GitHub PR 交付插件

**关联**：B1/REQ-64、B1/REQ-65、B1/REQ-66、B1/REQ-67、B1/REQ-68 → B1/TASK-15 → B1/CODE-O。

交付是普通 Agent，不在内核建交付状态机。稳定事件按 PR 分事项；可信提交状态与审查账本绑定轮次和来源。只审查、撤回、退回、仅合并、无需发布和 live 必须区分。真实业务副作用以 GitHub 读回为准。

**代码落点**：[plugins/delivery/delivery.ts](../../../plugins/delivery/delivery.ts)、[plugins/delivery/run.ts](../../../plugins/delivery/run.ts)、[plugins/delivery/agent.ts](../../../plugins/delivery/agent.ts)、[plugins/delivery/input.ts](../../../plugins/delivery/input.ts)、[plugins/delivery/poll.ts](../../../plugins/delivery/poll.ts)、[plugins/delivery/project.ts](../../../plugins/delivery/project.ts)、[plugins/delivery/review.ts](../../../plugins/delivery/review.ts)、[plugins/delivery/review-ledger.ts](../../../plugins/delivery/review-ledger.ts)、[plugins/delivery/review-policy.ts](../../../plugins/delivery/review-policy.ts)、[plugins/delivery/handoff.ts](../../../plugins/delivery/handoff.ts)、[plugins/delivery/github.ts](../../../plugins/delivery/github.ts)、[plugins/delivery/action.ts](../../../plugins/delivery/action.ts)、[plugins/delivery/action.yml](../../../plugins/delivery/action.yml)。

**函数 / 模块入口**：`deliver`、`poll`、`settings`、`reviewDelivery`、`decideShip`。
**行为证据**：[tests/delivery.test.ts](../../../tests/delivery.test.ts)、[tests/delivery-review.test.ts](../../../tests/delivery-review.test.ts)、[tests/delivery-input.test.ts](../../../tests/delivery-input.test.ts)、[tests/delivery-run.test.ts](../../../tests/delivery-run.test.ts)、[tests/delivery-ledger.test.ts](../../../tests/delivery-ledger.test.ts)、[tests/delivery-action.test.ts](../../../tests/delivery-action.test.ts)、[tests/delivery-scope.test.ts](../../../tests/delivery-scope.test.ts)、[tests/delivery-handoff.test.ts](../../../tests/delivery-handoff.test.ts)。
**核对要点**：head/main 变化不误合并；丢失合并响应先读回；只审查不合并；文档后续提交不掩盖未发布代码。

### PLAN-§16 场景模板与浏览器交接

**关联**：B1/REQ-69、B1/REQ-70、B1/REQ-71、B1/REQ-72 → B1/TASK-16 → B1/CODE-P。

每个 recipe 自带规则、必要脚本、任务 Skill 和启用说明。浏览器入口由可信配置指定，任务与网页内容不能改变目标或允许操作；受阻时主任务 ask 后结束，回答后只继续剩余步骤。

**代码落点**：[examples/recipes/](../../../examples/recipes/)、[examples/skills/](../../../examples/skills/)、[examples/watch-github-issues.sh](../../../examples/watch-github-issues.sh)、[examples/worker.sh](../../../examples/worker.sh)、[examples/channel.sh](../../../examples/channel.sh)、[examples/plugins/](../../../examples/plugins/)、[skills/agent-autoloop/SKILL.md](../../../skills/agent-autoloop/SKILL.md)。

**行为证据**：[tests/recipes.test.ts](../../../tests/recipes.test.ts)、[tests/business-recipes.test.ts](../../../tests/business-recipes.test.ts)、[tests/browser-handoff.test.ts](../../../tests/browser-handoff.test.ts)、[tests/examples.test.ts](../../../tests/examples.test.ts)、[tests/example-plugins.test.ts](../../../tests/example-plugins.test.ts)。
**核对要点**：网站失败只首次唤醒、恢复结案；飞书快照关联原事件；浏览器任务保持原会话，不重复委派。

### PLAN-§17 日志投影、兼容性与工程验证

**关联**：B1/REQ-73、B1/REQ-76、B1/REQ-78 → B1/TASK-17 → B1/CODE-Q。

commitLatest 在同一写事务内取得最新视图；commit 只追加事实，evolve 独占持久投影。WAL+FULL 逐笔提交；格式变化逐条重建而非业务重放。工程验证沿用日常、模拟、穷举、强杀、变异和跨平台 CI，不增加第二套内核或达标线。

**代码落点**：[kernel/store.ts](../../../kernel/store.ts)、[kernel/evolve.ts](../../../kernel/evolve.ts)、[kernel/query.ts](../../../kernel/query.ts)、[kernel/credentials.ts](../../../kernel/credentials.ts)、[sim/](../../../sim/)、[scripts/](../../../scripts/)、[tests/](../../../tests/)、[.github/](../../../.github/)、[package.json](../../../package.json)、[package-lock.json](../../../package-lock.json)、[tsconfig.json](../../../tsconfig.json)、[.gitignore](../../../.gitignore)、[.gitattributes](../../../.gitattributes)、[CONTRIBUTING.md](../../../CONTRIBUTING.md)。

**函数 / 模块入口**：`commit`、`commitLatest`、`evolve`、`openView`、`rebuild`。
**行为证据**：[tests/kernel.test.ts](../../../tests/kernel.test.ts)、[tests/projection.test.ts](../../../tests/projection.test.ts)、[tests/format.test.ts](../../../tests/format.test.ts)、[tests/purity.test.ts](../../../tests/purity.test.ts)、[tests/ci-scope.test.ts](../../../tests/ci-scope.test.ts)、[tests/simulation.test.ts](../../../tests/simulation.test.ts)、[sim/crash.test.ts](../../../sim/crash.test.ts)、[sim/mutation.test.ts](../../../sim/mutation.test.ts)。
**核对要点**：已确认序号与重建投影一致；投影失败回滚日志；指纹仅受持久格式材料影响；依赖无运行时包。

### PLAN-§18 文档基线与追溯入口

**关联**：B1/REQ-75 → B1/TASK-18 → B1/CODE-R。

需求、业务、技术、部署、使用和协议各自保持职责。基线 spec 和任务表明确标为事后整理，关联现有需求及固定实现，不冒充历史实施批准；原审阅报告保留当时状态和验证窗口。页面计划单列，未完成设计不伪造为本次交付。

**代码落点**：[docs/](../../../docs/)、[README.md](../../../README.md)、[README.en.md](../../../README.en.md)、[AGENTS.md](../../../AGENTS.md)、[INSTALL.md](../../../INSTALL.md)、[SECURITY.md](../../../SECURITY.md)、[LICENSE](../../../LICENSE)、[skills/agent-autoloop/SKILL.md](../../../skills/agent-autoloop/SKILL.md)。

**行为证据**：本文和关联 plan 的完整映射、本地引用及文档结构检查；不冒充运行测试。
**核对要点**：每个独立验收条目有来源、设计节、任务和实现证据；本地链接、标题锚点、UTF-8 及围栏有效；不签发未完成的 DELIVER。


## 失败、权限与状态限制

- 事件和回执去重不等于外部动作恰好一次；业务 Skill 必须核对外部当前状态。
- done 只结束一轮；卡片 decided 只记录决定，未必执行完成；事项由轮次、卡片和复查共同表达。
- 续接已开始后失败必须留在原会话重派，不能换新会话重做；进程查询失败不等于进程退出。
- 临时失败、需要本人恢复的阻塞、额度限制和本人停止分别处理，不能统一变成自动重试。
- askFirst 和 access 映射不能替代系统隔离；普通卡片链接持有者可以回答该卡片。
- 本人 MCP 不能放进普通 worker 轮次的用户级配置；交付审查不能执行 PR 中的任何程序或设置。

## 测试和证据状态

[2026-11-23 审阅记录](../../06-交付审计/2026-11-23-文档审阅.md)提供前次实际运行证据：97 个快速测试文件、851 项，830 通过、21 跳过、0 失败；类型和格式各一项通过。

该轮测试的原代码基准是 62feecc7dbf9e8f5da3c1249bb9f5b84459d576b；到本基线 71cc1d2 的实现、测试、安装脚本及依赖清单没有变更，区别为文档。证据可追溯，但不能谎称本次重新执行了应用测试。

20 个 Windows 专用场景和 1 个真实 fish 场景未在本机验证；未运行全量模拟、穷举、nightly、安装冒烟、真实账号、线上发布或性能实测。本次只新增资料，不把上述未运行项勾成完成。

## 待正式复核

1. 两份基线及索引写入并做内容和链接校验。
2. 经用户确认提交，使 spec、plan、需求和代码处于同一固定审查提交。
3. 从该提交重跑六项主链检查、五份 reference 对账和历史遗留核对。
4. 按实际发现及缓解决定 Verdict；材料存在本身不是 DELIVER 条件。

基线中发现或保留的差距仍要如实报告：现有运行日志尚未包含全局目标的全部字段，页面仍未实现，真实平台及跨平台验收未完成。本次不以补写文档把这些状态改成已完成。
