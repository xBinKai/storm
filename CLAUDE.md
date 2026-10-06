# CLAUDE.md

## 项目定位

本项目是 **stanford-oval/storm 的二次开发**，目标：打造面向专业投资机构研究员（买方/卖方分析师）的 **LLM/Agent 驱动的行业研究工作流**——从检索、分析到产出带引用的行业/公司研报。

- 上游 STORM 是通用 deep-research 报告生成器（dspy 四模块：知识整理/大纲生成/文章生成/文章润色），本项目在其骨架上做金融领域与中国市场适配
- **上游已静默（2025-09-30 后无 push），fork 即自立门户**——不要期待上游修复，改核心模块时以本地代码为准

## 市场与形态口径

- **A股/港股/中国为第一优先**，通用市场能力保留（上游原生）
- 产品形态 = **行业研究工作流**（检索 → 分析 → 带引用研报），不是每日盯盘/自动交易

## 奠基决策（2026-10-05，四候选对比调研后裁定）

选型报告（8 项目对比、三票对抗验证、含被证伪主张警示）：[`docs/investigations/llm-research-framework-survey-2026-10-05.md`](docs/investigations/llm-research-framework-survey-2026-10-05.md)

选 STORM 的理由：研究流水线本体是二开中最难自建的部分（对手 daily_stock_analysis 的核心是盯盘形态，恰是要拆掉的）；STORM 扩展点与机构需求精确对位：

- **VectorRM + 本地 Qdrant**：巨潮年报/券商研报私有语料 grounding 的现成入口
- **10+ 检索器统一为 dspy.Retrieve 子类、同构 forward 接口**（`knowledge_storm/rm.py`）：新增检索器 = 照抄一个类
- **litellm 逐组件换模**（`knowledge_storm/lm.py`）：DeepSeek/通义/内网模型随意配

## 落地路线

1. 写 `CninfoRetriever`——巨潮年报检索（端点调用参考无许可 PoC financial-poc 的 `report_downloader.py`，重写不抄文件）
2. VectorRM + 本地 Qdrant 灌年报/研报库；**embedding 必须换中文可用模型**（默认 HuggingFace embedding 中文不行，已验证的坑）
3. litellm 配 DeepSeek/通义；AkShare/Tushare 行情适配参考 daily_stock_analysis（MIT，保留署名）
4. FinGLM 的 1 万人工标注 QA 做研报质量评测基准（语料仅评测用，其代码无许可不可用）
5. 自建巨潮/券商研报摄取层——这是全开源生态的系统性缺口，无现成可商用实现

## 红线与注意

- **许可证**：上游 MIT；衍生自他项目的代码（daily_stock_analysis 的 Apache-2.0 适配层、financial-poc 的端点逻辑）一律重写实现并保留应尽的声明，不直接复制无许可代码
- **引用审计是必做项**：deep research agents 的引用幻觉有实证风险（内联引用事实核验通过率仅 39-77%，arXiv:2605.06635）——机构场景下报告引用须可溯源
- **数据合规未尽调**：Tushare/AKShare 数据用于机构对外研报分发的合规边界未验证，对外发布前需自行尽调
- 报告中的活动度数字（stars/push 日期）均为 2026-10-05 快照，引用须标注日期
