# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个 Claude Code Skill（非代码项目），用于财务费用异常识别。Skill 由纯 Markdown 规则文件组成，没有可执行代码、构建系统或依赖管理。

## 架构

**入口文件：** `SKILL.md` — 定义 skill 的 frontmatter 元数据、触发模式判断逻辑和三步执行流程（识别输入 → 并行五维度检测 → 汇总报告）。

**五维度检测规则（`rules/` 目录）：** 各规则文件独立运行，互不依赖。每个文件定义触发条件、风险等级阈值和输出格式：
- `amount.md` — 金额异常（超标准上限、整数金额疑点、超历史均值）
- `time.md` — 时间异常（非工作日、法定假日、深夜消费、发票日期不符）
- `frequency.md` — 频次异常（重复消费、拆单规避审批、相同金额重复）
- `compliance.md` — 合规检查（差旅标准超标、费用类型与商户不匹配、超单次上限）
- `invoice.md` — 发票检测（重复发票号、格式异常、连续发票号）

**输出模板：** `output-template.md` — 定义两种输出格式（自动节点模式/交互式模式）和风险等级定义（高/中/正常）。综合风险等级取所有命中维度中的最高级别。

**评估与测试：**
- `evals/evals.json` — 4 个自动化评估用例，含 prompt、expected_output 和 expectations 数组
- `tests/validation-scenarios.md` — 5 个手动验证场景

## 关键设计约定

- 两种触发模式：**自动节点模式**（结构化 JSON 输入，严格格式输出）和**交互式模式**（自然语言输入，对话式输出）
- 必须字段：amount、date、merchant、expense_type；缺失时追问一次，不重复追问
- 可选字段缺失时：对应维度静默跳过，在报告末尾注明"因信息不足未作检测"
- 规则文件中的阈值标注"可由企业覆盖配置"，但当前无配置机制，均为硬编码默认值
- 商户名相似判断：前 4 个字相同即视为同一商户（用于拆单检测）
