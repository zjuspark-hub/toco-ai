<h2 align="center">
  <br>
  <img height="80" alt="TocoAI" src="https://github.com/user-attachments/assets/b9ad1054-a73c-4b20-a36e-183ee1923f23" />
  <br>
  TocoAI：基于 DSL-Spec 的服务端 Harness Engineering
</h2>

<div align="center">

[![文档](https://img.shields.io/badge/文档-toco.ai%2Fdocs-brightgreen?style=flat&color=%2373DC8C)](https://toco.ai/docs)
[![开源协议](https://img.shields.io/badge/License-Apache%202.0-blue?style=flat&color=%234B78E6)](LICENSE)
[![技术栈](https://img.shields.io/badge/Stack-Java%20%7C%20Spring%20Boot-orange?style=flat)](https://toco.ai)
[![社区](https://img.shields.io/badge/加入社区-Discord-purple?style=flat&color=%23FA9BFA)](https://toco.ai/discord)

</div>

<br>

TocoAI 是服务端研发的 **Harness Engineering** 方案。Spec 不应是一次性生成代码的文本，而是**控制系统的结构层**。我们用 DSL-Spec 约束大模型生成，并通过建模引擎稳态渲染结构性，让 AI 始终在人的设计意图下工作。

<div align="center">

<img src="https://placehold.co/860x400/f8f9fa/333333?text=TocoAI+%E6%9E%B6%E6%9E%84%E5%9B%BE%EF%BC%9A+%E9%9C%80%E6%B1%82%E6%96%87%E6%A1%A3+%E2%86%92+DSL-SPEC+%E2%86%92+%E7%BB%93%E6%9E%84%E6%80%A7%E4%BB%A3%E7%A0%81+%2B+%E4%B8%9A%E5%8A%A1%E9%80%BB%E8%8F%9C+%E2%86%92+%E4%BA%A4%E4%BB%98%E7%B3%BB%E7%BB%9F" alt="TocoAI 工作流" width="100%"/>

</div>

<br>

> *"形式化符号是排除各种胡说八道的极其有效的工具。自然语言的自然性，不过是我们能轻松地用它说出那些荒谬性并不显而易见的话。"*
> — Edsger Dijkstra, EWD667, 1978

## 快速开始

- [安装与部署 →](https://toco.ai/docs/getting-started)
- [DSL 语法参考 →](https://toco.ai/docs/dsl)
- [建模引擎介绍 →](https://toco.ai/docs/engine)

## Harness Engineering 思路

服务端系统需要长时间维护，更像是建造一座大厦，而不是 3D 打印。我们用**建筑思维**来构建：

- 我们需要精准的设计图纸——于是定义了一套类 DDD、符合 CQRS 的 **DSL-SPEC**
- 图纸有了，我们更需要一致且可维护的代码结构——于是开发了**建模引擎**，稳态还原结构性代码，这部分在复杂项目中占到约 **80%**
- 最后在结构稳固、接口齐全的大楼里，用 AI 完成室内装修——那些难以 DSL 化的 if/else 业务逻辑

<div align="center">
<img src="assets/tocoai-arch.png" alt="TocoAI 架构图" width="100%"/>
</div>

## 核心组件

### 1. DSL-SPEC

需求和架构设计统一表达为结构化的 DSL Spec，作为整个系统的 **Single Source of Truth**。DSL 人类可读，机器可解析。它不是配置文件——**它是可执行的架构意图**。

|  | 自然语言 Spec | 程序式 IaC | **DSL-SPEC** |
|--|--------------|-----------|------------|
| **含义明确、可读性强** | 人人能读，但歧义不报错，只在生产事故里现身 | 精确，但需理解执行流程才能看懂意图 | ✅ 像自然语言一样可读，像程序一样精确，模糊即编译错误 |
| **细节清晰（What + How）** | 只有模糊的 What，How 全靠 AI 自由发挥 | 只有 How，架构意图藏在执行逻辑里无法单独审查 | ✅ What（数据意图）与 How（结构约束）同时显式表达 |
| **可校验、易维护（变更可查）** | 机器无法校验，改一个字段靠人肉追踪影响范围 | 可执行但无独立意图层，变更影响面需依赖分析工具 | ✅ 机器可解析可校验，改 DSL 引擎自动级联同步所有受影响结构 |
| **始终和代码一致** | 启动时准确，三个月漂移，六个月成历史文献 | 代码即实现，但架构意图随迭代逐渐丢失 | ✅ Spec 与代码的关系永远是 `=`，不是 `≈` |

DSL 覆盖后端系统的完整设计层级：

- **领域模型**：Entity / Relation
- **聚合对象**：BO
- **数据传输层**：DTO / VO
- **查询方案**：ReadPlan / WO
- **写入方案**：WritePlan
- **服务接口**：API / RPC / Service

一个 DTO 的 DSL 定义，自动生成：

```
UserDetailDto.java                (~60 Lines)
UserDetailDtoManager.java         (~25 Lines)
UserDetailDtoManagerImpl.java
UserDetailDtoConverter.java       (~80 Lines)
UserDetailDtoService.java         (~70 Lines)
UserDetailDtoDataAssembler.java
```

[查看完整 DSL 语法文档 →](https://toco.ai/docs/dsl)

### 2. 建模引擎

引擎覆盖全部结构性代码的生成：

- 分层骨架（`persist` / `manager` / `service` / `entrance`）
- 接口契约与数据模型
- CQRS 命令查询拆分
- 跨层转换器（`DtoConverter` / `VoConverter`）
- 数据拼装器（`DataAssembler`）
- 聚合写链路（`BoService`）

**开发者只需关注剩余 20% 的业务逻辑**，Review 成本骤降一个数量级。

建模引擎的本质，是将软件工程的质量控制**左移到"设计时"**。结构性代码由引擎确定性渲染，而非由大模型逐次生成——这从根本上消除了 AI 的随机错误和架构漂移。无论项目迭代多少轮、经手多少人，基础结构始终和设计保持一致，不会随时间腐化。

[查看引擎机制详解 →](https://toco.ai/docs/engine)

### 3. 标准的 Human-in-Loop

AI 从来不是神。对于长时间影响系统的领域划分、数据结构定义、接口描述、读写方案（事务约束、操作性能），我们提供 AI 修改与手动修改相结合的标准手段。

**让 AI 始终是人的助理，而不是主人。**

## 与 Cursor / Claude Code 的对比

Cursor、Claude Code 是优秀的工具，我们不是它们的替代品。

|  | Cursor / Claude Code | TocoAI |
|--|----------------------|--------|
| **定位** | 对话式编程助手 | 后端工程 Harness |
| **代码来源** | LLM 按 prompt 生成 | DSL → 引擎确定性生成（80%）+ LLM 实现业务 |
| **架构一致性** | 随迭代漂移 | DSL 锁定，不随时间和人员变化 |
| **变更方式** | 重新描述，人肉追踪影响范围 | 改 DSL，引擎自动级联同步 |
| **Review 成本** | 全量代码 | 只审 20% 业务逻辑 |
| **知识沉淀** | 对话用完即丢 | DSL 即文档，可传承 |
| **适用场景** | 原型、碎片化编码 | 复杂后端系统、长期多人协作 |

> **一句话：** 写一个能跑的系统，用 Cursor。建一个能活三年的系统，用 TocoAI。

## 核心优势

**AI 生成的代码，你敢直接上线吗？**
大多数团队 AI 代码采纳率不到 20%，剩下的时间在 Review、返工、祈祷没有漏网的坑。TocoAI 的 80% 结构性代码由引擎确定性渲染，不是 AI 猜的——这部分你不需要 Review，因为它就是设计的忠实还原。

**六个月后，你还认识自己的系统吗？**
AI 辅助开发的系统，架构在第一次迭代后开始漂移，半年后新人看不懂，一年后原作者也不敢动。DSL 是系统唯一的真相来源，引擎保证每一次生成的结构和设计一致——不管迭代了多少次，换了多少人。

**核心成员离职，带走的不只是人**
口传心授的架构决策、散落在聊天记录里的设计意图、只有老人才知道的"别动那块代码"——这些知识住在人头脑里，不住在系统里。DSL 把架构意图固化成机器可读的文档，交接靠读 DSL，不靠问人。

**人越加越乱，是因为规范只在嘴上**
Code Review 约定靠默契，代码风格靠个人自觉，AI 生成质量靠 prompt 技巧——团队一大，这套东西就撑不住。TocoAI 把规范烧进引擎，新人第一天产出的结构和老人一样，团队越大，优势越明显。

## 真实案例

### 大型医院 HIS 系统

**背景：** 新一代全院管理系统，120+ 核心模块，200+ 业务流程，多团队并行开发。

**难点：** 医疗业务零容错，多人跨时间协作极易产生逻辑漏洞和技术债。

**结果：** 架构规范统一执行，生成代码准确率显著提升，整体项目 AI 代码采纳率近 **97%**，交接时新成员可快速上手项目全局。

[查看案例详情 →](https://toco.ai/cases/his)

### 金融保证金支付系统重构

**背景：** 重构遗留系统，原技术栈为 SQLServer 2008 + 大量存储过程 + 多个三方中间件。

**难点：** 业务流程散落，数据和业务关联关系不明，性能瓶颈明显。

**结果：** 通过 DSL 重新建模，业务关联关系显式化，端到端流程可追溯，系统可在 TocoAI 框架下持续迭代。

[查看案例详情 →](https://toco.ai/cases/finance)

## 适用场景与局限

当前 DSL 只适合描述**关系型数据库驱动的后端系统**。Harness Engineering 的思路本身是通用的，但需要针对不同领域重新定义 DSL——我们期待社区在嵌入式、前端组件、基础设施配置等场景延伸这个思路。

**老项目兼容：** 可以在老项目中开发新模块，存量遗留代码的整体接管不在覆盖范围内。正确的路径是用新模块承接新需求，逐步把老逻辑迁移进 DSL 管辖范围——**重构是 AI 时代的日常工作**。

## 社区与参与

- 访问 [toco.ai](https://toco.ai) 获取完整文档
- 加入 [Discord 社区](https://toco.ai/discord) 提问、讨论功能和分享实践
- 在 [GitHub Issues](https://github.com/tocoai/toco/issues) 提交 Bug 报告或功能建议
- 我们**愿意开源**，和社区一起共建新的 DSL 表达能力

## 参与贡献

浏览现有的 [Issues](https://github.com/tocoai/toco/issues) 和 [Pull Requests](https://github.com/tocoai/toco/pulls)，或 [提交新 Issue](https://github.com/tocoai/toco/issues/new)。

没有完美的技术，只有适应的场景。毕竟 AI 改变研发才一年，但软件开发本身已经有七、八十年的历史。

[贡献指南 →](CONTRIBUTING.md)
