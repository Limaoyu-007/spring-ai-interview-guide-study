# 简历上传与异步分析链路总结

用户在前端上传简历时，后端 `ResumeController` 通过 `MultipartFile` 接收文件请求，并把具体业务交给 `ResumeUploadService` 处理。

`ResumeUploadService` 首先会对文件进行基础校验，包括文件是否为空、大小是否超限、文件类型是否支持。随后系统会使用 Apache Tika 相关能力解析简历内容，将 PDF、DOCX、TXT 等格式的文件提取成纯文本。

在解析成功后，原始简历文件会被上传到对象存储中，数据库中则会保存一条 `ResumeEntity` 记录。`ResumeEntity` 主要保存文件 hash、原始文件名、文件大小、对象存储地址、解析后的简历文本以及分析状态。

新上传的简历初始状态是 `PENDING`，表示简历已经入库，但 AI 分析任务还没有真正开始执行。

接下来，`AnalyzeStreamProducer` 会把 `resumeId`、简历文本内容和 `retryCount` 封装成消息，写入 Redis Stream。这样上传请求就不需要同步等待 AI 大模型返回结果，而是可以快速返回给前端。

应用启动后，`AnalyzeStreamConsumer` 会在后台线程中持续监听 Redis Stream。当它读取到简历分析任务后，会先把简历状态更新为 `PROCESSING`，表示任务正在处理中。

真正的 AI 分析逻辑由 `ResumeGradingService` 完成。它会基于解析出来的简历文本调用 Spring AI 和大模型，生成总分、各维度评分、简历摘要、优点和改进建议。

AI 分析完成后，系统会将分析结果保存为 `ResumeAnalysisEntity`，并与原来的 `ResumeEntity` 建立关联。保存成功后，简历状态会从 `PROCESSING` 更新为 `COMPLETED`。

如果分析过程中发生异常，Consumer 会根据 `retryCount` 判断是否重新入队重试。如果超过最大重试次数仍然失败，则会把状态更新为 `FAILED`，并记录失败原因。

最后，Consumer 会对当前 Redis Stream 消息执行 `ACK`。这里的 `ACK` 表示当前消息处理流程已经结束，并不等同于业务一定成功；如果失败但已经重新入队或标记失败，原消息同样会被 ACK。
