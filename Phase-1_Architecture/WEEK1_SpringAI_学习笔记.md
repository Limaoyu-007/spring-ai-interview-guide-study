# Spring AI 项目实战学习笔记（第 1 周）

> 项目：`interview-guide`
> 
> 学习者背景：大二，具备 Java 后端基础
> 
> 学习目标：借助真实项目掌握 Spring AI、RAG、异步任务链路，并形成可面试表达的项目理解

---

## 1. 第 1 周目标与完成情况

### 本周目标
1. 跑通项目（前后端 + PostgreSQL + Redis + MinIO）
2. 从用户视角完整体验核心功能
3. 梳理三条核心业务链路
4. 理解 Producer/Consumer 异步处理模型
5. 产出架构图与时序图

### 完成情况（已完成）
- [x] 后端、前端及依赖服务全部启动成功
- [x] 简历上传与分析流程验证成功
- [x] 模拟面试（出题、答题、评估）流程体验完成
- [x] 知识库上传与问答流程体验完成
- [x] 修复对象存储连接问题（`localhost:9000` refused）
- [x] 梳理三条主链路
- [x] 理解异步状态流转：`PENDING -> PROCESSING -> COMPLETED/FAILED`
- [x] 完成两张图：系统架构图 + RAG 时序图

---

## 2. 项目技术栈理解（第 1 周版）

## 后端核心
- `Spring Boot 4`
- `Java 21`
- `Spring AI`
- `Spring Data JPA`
- `Redis + Redis Stream`
- `PostgreSQL + pgvector`
- `MinIO/RustFS (S3 兼容)`
- `Apache Tika`

## 前端
- `React + TypeScript + Vite`

## 构建工具
- 本项目使用 **Gradle**（不是 Maven）
- 关键文件：
  - `app/build.gradle`
  - `gradle/libs.versions.toml`
  - `settings.gradle`

---

## 3. 关键概念澄清

## 3.1 为什么没有 `pom.xml`
因为项目使用 Gradle，不使用 Maven。依赖和版本管理由 `build.gradle` + `libs.versions.toml` 完成。

## 3.2 `spring-boot-starter-data-jpa` 是什么
- JPA：Java 持久化规范
- Hibernate：JPA 常见实现
- 作用：通过 `Entity + Repository` 实现对象与表映射，减少手写 SQL

对比习惯：
- MyBatis：`Mapper + SQL`（手写 SQL）
- JPA：`Repository + Entity`（框架生成常见 SQL）

## 3.3 Tika 是什么
`Apache Tika` 用于解析 PDF/DOCX/TXT 等文件并提取纯文本，供后续 AI 分析或向量化使用。

## 3.4 为什么 PostgreSQL 而不是 MySQL
核心不是“谁更好”，而是项目需要 `pgvector` 进行向量检索，PostgreSQL 在该场景下集成方便。

---

## 4. 三条核心业务链路（第 1 周重点）

## 4.1 链路一：简历上传与分析

**入口**：`POST /api/resumes/upload`

**主流程**：
1. 接收文件
2. 文件校验（大小/类型）
3. 去重（文件 hash）
4. 文本解析（Tika）
5. 原文件上传对象存储（S3 接口）
6. 保存简历记录到数据库（状态先 `PENDING`）
7. 发送分析任务到 Redis Stream（异步）
8. 返回 `resumeId + analyzeStatus`

**关键点**：
- 接口不阻塞等待 AI 完成
- 前端通过状态轮询获取最终结果

---

## 4.2 链路二：模拟面试

**入口示例**：
- 创建会话：`POST /api/interview/sessions`
- 提交答案：`POST /api/interview/sessions/{sessionId}/answers`

**主流程**：
1. 创建会话时先检查是否有未完成会话（避免重复）
2. 基于简历（+历史问题）生成面试题
3. 题目写入缓存并持久化
4. 用户提交答案（按 `sessionId + questionIndex`）
5. 更新题目答案与当前进度，写回缓存/数据库
6. 最后一题完成后，异步发送“评估任务”
7. Consumer 完成评估并保存报告

**关键点**：
- 答题与评估解耦
- 评估重活异步处理

---

## 4.3 链路三：知识库（RAG）

### 文档上传链路
**入口**：`POST /api/knowledgebase/upload`

流程：校验 -> 去重 -> 解析 -> 存储 -> 入库(`PENDING`) -> 向量化任务入队（异步）

### 问答链路（A/B 区分）

#### A：知识库查询接口（偏“检索问答”）
- `POST /api/knowledgebase/query/stream`
- 特点：更关注检索与回答本身

#### B：RAG 聊天接口（偏“会话聊天”）
- `POST /api/rag-chat/sessions/{sessionId}/messages/stream`
- 特点：有会话、消息历史、占位消息与流结束回填

**A/B 核心区别**：
- A：问一次答一次（轻会话/无会话）
- B：完整聊天产品形态（强会话）

---

## 5. 异步模型（Producer / Consumer）

## 5.1 本项目里的异步对象
- 简历分析：`AnalyzeStreamProducer/Consumer`
- 知识库向量化：`VectorizeStreamProducer/Consumer`
- 面试评估：`EvaluateStreamProducer/Consumer`

## 5.2 通用抽象
- `AbstractStreamProducer`
- `AbstractStreamConsumer`
- `AsyncTaskStreamConstants`

## 5.3 Consumer 会不会一直消费？
会。应用启动后，Consumer 在后台线程执行循环轮询，持续从 Stream 消费消息。

## 5.4 状态流转
通常为：
- 入队前后：`PENDING`
- 开始处理：`PROCESSING`
- 处理成功：`COMPLETED`
- 失败超重试：`FAILED`

## 5.5 ACK 的准确理解
- ACK 不是“业务成功”
- ACK 表示“这条消息已处理结束（成功或失败分支）”
- 本项目在失败重试时会“重新入队新消息 + ACK 原消息”

---

## 6. 本周踩坑记录

## 问题：上传简历时报对象存储连接拒绝
- 现象：`Connect to localhost:9000 failed: Connection refused`
- 原因：后端访问的对象存储地址不可达（MinIO 未启动或 endpoint 配置不一致）
- 修复：
  1. 启动 MinIO
  2. 统一 endpoint/ak/sk/bucket 配置
  3. 重新上传验证
- 结论：对象存储链路是上传流程关键依赖

---

## 7. 对项目代码风格的阶段性理解

第 1 周感受“难”的原因：
1. 分层较多（Controller/Service/Repository/Infra）
2. 抽象基类（模板方法）较多
3. 新写法较多（`record`、`Flux`、`Optional`、泛型 DTO）
4. 同时涉及 AI、存储、缓存、DB、异步，多技术叠加

当前正确学习法：
- 先看主流程“动词链路”
- 再看每一步实现
- 暂不追求一次看懂所有工具类

---

## 8. 我现在已经能讲清楚的内容

1. 项目为什么要用异步队列
2. 简历/面试/知识库三条业务主链路
3. A/B 两套问答接口的产品定位区别
4. Producer/Consumer 在本项目里的职责分工
5. ACK、重试、状态流转的基本语义

---

## 9. 第 2 周学习计划（下一步）

## 目标
进入 Spring AI 核心能力学习：
- Prompt 组织
- 结构化输出
- RAG 参数理解与实验

## 建议顺序
1. `common/ai/StructuredOutputInvoker.java`
2. `modules/interview/service/InterviewQuestionService.java`
3. `modules/knowledgebase/service/KnowledgeBaseQueryService.java`

## 第 2 周最小产出
- 一份“结构化输出”调用链说明
- 一次检索参数实验（`topK/minScore`）记录
- 一段 3~5 分钟面试表达稿

---

## 10. 本周一句话总结

第 1 周已从“能跑项目”升级到“能讲清主链路与异步架构”，下一步进入 Spring AI 核心调用与参数调优阶段。
