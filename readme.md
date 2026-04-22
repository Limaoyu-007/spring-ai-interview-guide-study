# 🚀 Spring AI 实战：Interview Guide 深度拆解与重构记录

> **“不只是把项目跑起来，而是把黑盒变成自己的武器库。”**

[![Java](https://img.shields.io/badge/Java-21-orange.svg)]()
[![Spring Boot](https://img.shields.io/badge/Spring%20Boot-4.x-brightgreen.svg)]()
[![Spring AI](https://img.shields.io/badge/Spring%20AI-Framework-blue.svg)]()
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-pgvector-blue.svg)]()
[![Redis](https://img.shields.io/badge/Redis-Stream-red.svg)]()

本仓库记录了一名 Java 后端学习者，对开源项目 `interview-guide`（基于大模型的智能面试平台）的源码级拆解、RAG 调优实战以及工程化落地全过程。

这不是一份简单的“代码搬运”，而是一份面向**“架构理解”**与**“面试表达”**的深度学习笔记。

---

## 🎯 核心目标：跨越“调包侠”的四层境界

我学习这个项目的目标，是完成以下四个层次的跨越：

- **🟢 第一层：会运行** —— 跑通前后端、PostgreSQL、Redis、MinIO 等完整中间件依赖，闭环核心功能。
- **🔵 第二层：会看懂** —— 拆解简历分析、模拟面试、RAG 问答三条主链路，理解 `Controller -> Service -> Repository` 的数据流转与异步状态机。
- **🟡 第三层：会讲清** —— 提炼技术选型背后的逻辑（为什么用 Redis Stream？为什么用 pgvector？），输出架构图与时序图。
- **🔴 第四层：会改造** —— 深入 Spring AI 底层，玩转 Prompt 模板、结构化输出（Structured Output），并能独立完成 RAG 检索参数（topK/minScore）的对比实验。

---

## 🏗️ 核心业务模块拆解

本项目是一个完整度极高的 AI 商业级落地 Demo，我主要聚焦于以下三大模块的底层拆解：

1. **📄 简历分析模块：** `Tika` 文本解析 -> 异步上传对象存储 -> AI 总结 -> `Redis Stream` 异步状态流转。
2. **🎙️ 模拟面试模块：** 动态 Prompt 组装 -> 结构化数据响应 -> 答题状态机流转 -> 异步评估报告生成。
3. **🧠 知识库 (RAG) 模块：** 文档 Chunking 与 Embedding -> 向量入库 -> 相似度检索 -> `ChatClient` 流式响应 (SSE) 与历史会话记忆。

---

## 🗺️ 学习路线与进度打卡

本项目拆解计划分为 4 个阶段，通过**“主链路优先 -> 局部深入 -> 工程打磨”**的策略逐步推进：

### 🏁 Phase 0: 基线建立 (100%)
- [x] 后端、前端及所有中间件全量启动
- [x] 解决首次启动环境踩坑（如 MinIO 端口拒绝连接）
- [x] 完成用户视角的三大核心业务闭环

### 🔍 Phase 1: 后端骨架与链路透视 (Current)
- [x] 梳理简历上传、模拟面试、知识库问答三条主链路
- [x] 理解基于 `Redis Stream` 的 Producer/Consumer 异步解耦模型
- [x] 产出：系统全局架构图、RAG 核心交互时序图
- [ ] *[📝 笔记：Week 1 核心架构与异步链路拆解](./Phase-1_Architecture/Week1_Overall_Architecture.md)*

### 🧠 Phase 2: Spring AI 核心能力深入 (Pending)
- [ ] 源码拆解：`ChatClient` API 调用与组装方式
- [ ] 源码拆解：Prompt Template 的动态构建
- [ ] 核心机制：深入探索 `StructuredOutputInvoker`（结构化输出）
- [ ] 核心机制：Advisor 拦截器与聊天上下文记忆机制

### ⚙️ Phase 3: RAG 调优与面试化打磨 (Pending)
- [ ] 向量化全流程剖析（分块、Embedding、存储）
- [ ] 对照实验：检索参数 `topK` / `minScore` 对 AI 回答质量的实测影响
- [ ] 最终沉淀：提炼 5 分钟项目面试讲稿与高频防追问 Q&A 手册

---

## 💡 我的“拆解”方法论

初涉复杂项目，容易溺死在源码细节中。我的行动准则是：

1. **AI 辅助探索：** 通过问答式学习（如 Cursor）建立认知框架：先问“做什么”，再问“怎么做”，最后探索“怎么优化”。
2. **拒绝盲目断点：** 遵循 `骨骼(pom) -> 血液(yml) -> 神经(Controller/Service) -> 肌肉(Prompt/SQL)` 的分层阅读法。
3. **无输出，不学习：** 每梳理完一个模块，必须落实为架构图、时序图或 Markdown 总结，将知识转换为可视化的技术资产。

---

## 📂 仓库目录导览

*(随着学习进度持续更新)*

```text
├── README.md                      # 你正在看的文件：项目主页与学习地图
├── Phase-1_Architecture/          # 第 1 阶段：架构剖析与异步链路
│   └── Week1_Overall_Architecture.md 
├── Phase-2_SpringAI_Core/         # 第 2 阶段：Spring AI 调用实战 (TODO)
├── Phase-3_RAG_Tuning/            # 第 3 阶段：RAG 检索增强参数实验 (TODO)
├── Phase-4_Interview_Q&A/         # 第 4 阶段：面试实战与防追问手册 (TODO)
└── assets/                        # 架构图、时序图及相关截图资料