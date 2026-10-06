# 第四轮调研：LLM/Agent 驱动的投资行业研究工作流开源框架

> **调研日期**：2026-10-05（所有 stars/forks/push 日期均当日实抓，漂移很快，引用须标注快照日期）
> **调研方法**：deep-research 工作流——5 路并行检索 → 19 个来源抓取 → 93 条主张提取 → 25 条三票对抗验证（**24 确认 / 1 证伪**）
> **需求口径**（三轮澄清后确定）：LLM/Agent 驱动的研究框架形态；A股/港股/中国 + 通用市场；技术栈不限
> **决策结论**：**选 [stanford-oval/storm](https://github.com/stanford-oval/storm) 作为二次开发底座**（已 clone 至 `~/Documents/storm`）

---

## 一句话结论

**没有任何单一项目同时满足「机构研究工作流 + 中国市场文档摄取（巨潮/券商研报）+ 可商用许可 + 活跃维护」**。中国原生框架全部倒在许可或成熟度闸门上；务实的机构路线是组合：STORM 做研报生成流水线 + daily_stock_analysis 做中国行情数据适配参考 + 自建巨潮采集层。

## 候选项目排名（按商用二次开发底座适合度）

| # | 项目 | 许可证 | 活跃度（2026-10-05 快照） | 商用二开判定 |
|---|------|--------|--------------------------|-------------|
| 1 | [ZhuLinsen/daily_stock_analysis](https://github.com/ZhuLinsen/daily_stock_analysis) | MIT ✅ | 65.9k stars，push 当日 | ✅ 可（嵌套 Apache-2.0 子系统需保留声明） |
| 2 | [stanford-oval/storm](https://github.com/stanford-oval/storm) | MIT ✅ | 31.6k stars，push 2025-09-30 | ✅ 可（**已选定**） |
| 3 | [virattt/ai-hedge-fund](https://github.com/virattt/ai-hedge-fund) | MIT ✅ | 63.9k stars，push 2026-10-02 | ✅ 可，但定位是架构模式供体 |
| 4 | [langchain-ai/open_deep_research](https://github.com/langchain-ai/open_deep_research) | MIT | 12.7k stars，**2026-08-21 已归档只读** | ⚠️ 仅可 fork 自持 |
| 5 | [hsliuping/TradingAgents-CN](https://github.com/hsliuping/TradingAgents-CN) | 混合（部分专有）❌ | 32.2k stars，push 2026-09-22 | ❌ app/frontend/core 禁商用 |
| 6 | [KylinMountain/TradingAgents-AShare](https://github.com/KylinMountain/TradingAgents-AShare) | PolyForm-NC ❌ | 851 stars，维护模式 | ❌ 新组件商用需作者许可 |
| 7 | [MetaGLM/FinGLM](https://github.com/MetaGLM/FinGLM) | 无许可证 ❌ | 休眠（2024-05 停更） | ❌ 仅数据集参考价值 |
| 8 | [Chai060120/financial-poc](https://github.com/Chai060120/financial-poc) | 无许可证 ❌ | 0 stars，个人 PoC | ❌ 仅代码参考价值 |

---

## 分项目详评（均为三票全票验证，标注票数与证据层级）

### 1. daily_stock_analysis — 中国数据覆盖最优，但形态不对口

- **架构**：「A股/港股/美股/日股/韩股/台股自选股智能分析系统」。数据层零配置内置 AkShare/Baostock/YFinance，token 型升级 Tushare/Longbridge/TickFlow 在依赖与配置层真实落地（`requirements.txt` 含 akshare>=1.12.0 / baostock / yfinance / tushare / longbridge / tickflow；`.env.example` 定义对应 token——代码级核验）。LLM 层经 litellm（钉 1.80.10+）支持 OpenAI 兼容端点/DeepSeek/通义/Claude/Gemini/聚合商/本地 Ollama，配 `docs/LLM_CONFIG_GUIDE.md`。
- **市场覆盖**：A股/港股为一等公民（README 逐字+示例代码 `600519,hk00700,AAPL` 佐证）；日/韩/台为 suffix-only 浅覆盖 MVP。
- **许可注意**：`src/services/screening` 为嵌套 Apache-2.0（衍生自 AlphaSift），需保留声明——通知义务而非 copyleft，不阻商用。`THIRD_PARTY_NOTICES.md` 已披露。
- **短板**：README 自警免费源受上游限流/接口变动影响；无巨潮/券商研报集成；**产品形态是每日盯盘分析而非行业研究/研报生成工作流，研究流水线层需全部自建**。
- 票数：3-0 ×3（claims 13-15 合并）

### 2. STORM — 最具扩展性的带引用研报流水线（已选定）

- **架构**：两阶段流水线（Pre-writing：联网检索收参考文献+生成大纲 → Writing：基于大纲+文献生成带引用全文），dspy 实现四大可定制模块（知识整理/大纲生成/文章生成/文章润色）——README 逐字+代码级核验。
- **检索层**：10 种可插拔后端（YouRM/BingSearch/VectorRM/SerperRM/BraveRM/SearXNG/DuckDuckGoSearchRM/TavilySearchRM/GoogleSearch/AzureAISearch），在 `knowledge_storm/rm.py` 代码级逐一确认（统一 dspy.Retrieve 子类、同构 forward 接口；验证人注：实为 11 个，含未列入 README 的 ArxivRM——多的方向不构成证伪）。
- **关键扩展点**：**VectorRM 支持自有文档 grounding**——官方示例 `run_storm_wiki_gpt_with_VectorRM.py` + 文档化 CSV 语料格式 + **本地 Qdrant 离线模式**——接入机构私有语料（巨潮年报/券商研报本地库）的现成入口。
- **模型层**：自 v1.1.0（2025-01-23 发布）经 litellm 集成，覆盖 litellm 全部 LLM/embedding 模型且各组件可配不同模型平衡成本；旧 per-provider wrapper 全部标 deprecated。
- **许可**：逐字标准 MIT（1091 字节，全树仅一份许可文件，无附加条款）。
- **短板**：零金融领域逻辑、零中国数据源，领域适配全部自建；VectorRM 默认 HuggingFace embedding 需换中文可用模型；Bing API 有弃用风险。
- **活跃度注意**：v1.1.1 发布 2025-09-29、最后 push 2025-09-30——快照日已静默约一年（未归档）。对 fork 自持影响小，但上游不会继续修 bug/跟进模型生态，fork 即自立门户。
- 票数：3-0 ×4（claims 2-5 合并，代码级核验）

### 3. ai-hedge-fund — 工程质量最高的架构模式供体

- **架构**：`hedge_fund/llm` 定义 runtime_checkable `LLMClient` Protocol + `make_llm()` 工厂（代码级核验+契约测试），支持 Anthropic/OpenAI（含 base_url 覆盖）/DeepSeek/Google/xAI/Kimi/TypeSafe。分析师角色/辩论式多智能体设计。
- **数字复核**：63,851 stars / 11,223 forks / 937 commits（Link header 分页精确计数），2024-11-29 创建，push 2026-10-02。
- **定位警示**：README 逐字自认教育性 PoC，「the system does not actually make any trades」，仅 paper trading + backtesting，无研报生成。**适合抄分析师角色与辩论编排架构，不作研究底座本体**。
- **注意**：v2 重写删减了 v1 的 Ollama/OpenRouter 本地推理 provider——影响离线部署。
- 票数：3-0 ×4（claims 6-9 合并）

### 4. LangChain Open Deep Research — 架构契合但已死

- **架构**（2-1 非全票通过，验证人评逐字支持、一票以「呈现已过时架构」反对）：LangGraph supervisor 将子问题分派给**隔离上下文窗口**的 sub-agents（各自跑用户可配置 search/MCP 工具循环），终稿由预生成 research brief 引导的单次 LLM 调用产出；明确定位可配置底座（模型/检索/MCP server 皆可插换）。
- **致命缺口**：仓库 2026-08-21 被归档只读（GitHub API archived=true 实证）。只能 fork 自持；LangChain 指向 Deep Agents 为后继路径（后继本身未独立验证）。
- 票数：2-1（claim 0）+ 3-0（claim 1）

### 5. TradingAgents-CN — 最完整 A股工作流，许可被阻断

- **架构**：TauricResearch/TradingAgents 中文增强衍生版（代码级确认派生关系）。角色化智能体（大盘/板块/市场/基本面/新闻分析师 + 乐观/审慎研究员）5 级可配深度辩论协作；原生 Tushare/AKShare/BaoStock 适配器（`app/services/data_sources/` 下三个 adapter 文件实证）。
- **许可阻断**（LICENSE 逐字）：混合授权——仅 `tradingagents/`、`cli/`、`docs/`、`tests/` 为 Apache-2.0；**`app/`、`frontend/`、`core/` 为专有 source-available，禁再分发与商业使用**；GitHub license=NOASSERTION。Word/PDF 报告导出、定时/批量分析为 Pro 付费功能。
- **商用路线**：谈商业许可，或仅复用其 Apache-2.0 目录。
- 票数：3-0 ×3（claims 10-12 合并）

### 6. TradingAgents-AShare — PolyForm-NC + 维护模式

- LICENSE 逐字：上游衍生核心 Apache-2.0，全部新组件（api/、frontend/ 等）及对核心的重大修改为 **PolyForm Noncommercial 1.0.0**（商用需作者明示许可）。
- 活跃度：851 stars / 328 commits，2026-03-02 创建、push 2026-09-18；末批提交全为 dependabot bump，最后 feature 提交停在 2026-07-20（约两月提交空窗）。
- 票数：3-0 ×2（claims 22-23 合并）

### 7. FinGLM — 无许可休眠，但语料资产独特

- SMP 2023 ChatGLM 金融挑战赛代码库：~70 commits、最后 push 2024-05-08、README 停更 2023-11；**无任何许可证**（全树 1384 路径扫描无项目 LICENSE），免责声明明言「一般不建议用于商业用途」。
- **独特资产**：11,588 份 A股年报 PDF（2019-2021、约 69GB，托管 ModelScope `chatglm_llm_fintech_raw_dataset`）+ 10,000 条人工标注 QA（初赛 5,000/复赛A 2,000/复赛B 2,000/复赛C 1,000，QA 在仓内 data 目录）。**仅作年报评测集/语料参考**。
- 票数：3-0 ×3（claims 16-18 合并）

### 8. financial-poc — 唯一的巨潮采集参考实现

- **唯一实现巨潮（cninfo）年报自动检索的项目**（tarball 代码级核验）：`report_downloader.py` 按公司名+年份调真实端点（`www.cninfo.com.cn/new/information/topSearch/query` 查 orgId、`/new/hisAnnouncement/query` 查公告、`static.cninfo.com.cn` 下载 PDF，含 A股类目码 `category_ndbg_szsh` 与沪/深/北市场路由）。
- 但 0 stars / 9 commits / 2026-08-12 最后 push，**无许可证**（README 自述「按需自行补充许可证」，法律默认保留所有权利）——商用阻断，仅作可运行参考实现（代码自认反爬脆弱）。
- 票数：3-0 ×3（claims 19-21 合并，含 tarball 代码级核验）

---

## 关键发现：中国市场文档数据缺口是系统性的

（多全票主张合成，confidence: medium——依赖 README/代码树级缺失检索，强但非穷尽）

- 成熟且许可干净的候选数据面**止步于行情/财务报表**：TradingAgents-CN 全树（1,881 文件）grep 巨潮/cninfo/研报/券商 = **0 命中**，16 万字用户手册亦 0 命中；daily_stock_analysis 依赖与配置层无任何 cninfo/研报集成（新闻/公告腿经东财，非巨潮非券商研报）。
- 巨潮摄取能力只存在于无许可 0-star 的 financial-poc；年报语料只存在于休眠无许可的 FinGLM（且仅 2019-2021 部分公司）。
- **即：面向 A股的机构研究栈必须自建巨潮/券商研报摄取层**，其中 STORM 的 VectorRM（本地 Qdrant 私有语料 grounding，CSV 入库格式）是所调查底座中最直接的私有语料接入扩展点。

## 决策：为什么选 STORM

选底座原则：**选「要造的东西的核心骨架最难自建」的项目**。

| | STORM | daily_stock_analysis |
|---|---|---|
| 核心资产 | 研究流水线本体（两阶段、带引用、四模块可定制）——正是要造的东西 | 自选股每日盯盘——恰是要拆掉的东西 |
| 缺的部分 | 中国数据源适配 | 研究流水线本体 |
| 缺的部分自建难度 | 低（AkShare/Tushare 适配是薄胶水，daily_stock_analysis 有现成参考） | 高（多阶段研究流水线架构是真正难的部分） |

STORM 扩展点与机构需求精确对位：VectorRM+本地 Qdrant = 巨潮年报/券商研报私有语料 grounding 现成入口；10 检索器统一 dspy.Retrieve 子类同构接口 = 写 `CninfoRetriever`/`AkShareRetriever` 照抄一个类；litellm 逐组件换模 = DeepSeek/通义/内网模型随便配。

诚实代价：上游 2025-09-30 后静默约一年——fork 自持影响小，但别指望上游修 bug。daily_stock_analysis 的持续维护数据源适配以**参考代码**方式获取（MIT，抄适配层+保留署名）。

**落地路径**：

```
1. Fork stanford-oval/storm（MIT，干净单许可）——已完成 clone 至 ~/Documents/storm
2. 写 CninfoRetriever（照 rm.py 现有 Retriever 类抄；cninfo 端点调用参考 financial-poc）
3. VectorRM + 本地 Qdrant 灌年报/研报库；embedding 换中文可用模型（默认 HF embedding 中文不行——已验证的坑）
4. litellm 配 DeepSeek/通义；AkShare/Tushare 适配参考 daily_stock_analysis
5. FinGLM 的 1 万 QA 做研报质量评测基准
```

---

## 保留意见（引用本报告前必读）

- **被证伪主张（0-3，不得引用）**：daily_stock_analysis 一条活动度叙述称「63,056 stars / 53,036 forks / 静默 7 周 / fork-to-run Actions 部署模式」——实测当日仍在 push（2026-10-05T07:52:27Z）、65.9k stars，该数字与「≈stars 的 forks」说法均错。
- Open Deep Research 架构主张以 2-1 非全票通过；「仓库已归档」与「Deep Agents 为后继」是验证人补充的机械证据，Deep Agents 本身未经独立验证。
- 机构级缺口角度只被部分覆盖：引用生成与私有化部署有证据（另两篇 arXiv：deep research agents 引用 URL 幻觉率升高、内联引用事实核验通过率仅 39-77%——引用审计是必做项），但**审计链路、Tushare/AKShare 数据用于对外研报的合规边界**本轮无存活主张评估，立项前需自行尽调（SEC/FINRA 立场：用 AI 不减免合规义务）。

## 未验证线索（预算内未过验证闸，值得二轮核查）

1. **GPT Researcher**（assafelovic/gpt-researcher）——Apache-2.0，某博客快照 ~28.9k stars、planner-executor 架构 + MCP；原始问题点名但未进验证集。
2. **FinRobot**（AI4Finance-Foundation）——Apache-2.0，V2 自称 9 agents + 7 条分析师管线（DCF/comps/LBO/DDM/IC memo）；需分辨开源版与 FinRobot Pro 功能面。
3. **MS-Agent FinResearch**（ms-agent）——五智能体 DAG 研报生成，金融数据腿原生覆盖 A股/港股/美股。
4. **LangChain Deep Agents**——Open Deep Research 归档后的官方后继。

## 主要来源

- https://github.com/stanford-oval/storm （primary，5 claims，全票）
- https://github.com/ZhuLinsen/daily_stock_analysis （primary，5 claims）
- https://github.com/virattt/ai-hedge-fund （primary，5 claims，全票）
- https://github.com/hsliuping/TradingAgents-CN （primary，5 claims）
- https://github.com/langchain-ai/open_deep_research + https://www.langchain.com/blog/open-deep-research （primary）
- https://github.com/MetaGLM/FinGLM 、https://github.com/Chai060120/financial-poc 、https://github.com/KylinMountain/TradingAgents-AShare （primary）
- 引用幻觉/合规切面：arXiv:2604.03173、arXiv:2605.06635、arXiv:2601.20727、FINRA AI 报告、mofo.com 合规提示
