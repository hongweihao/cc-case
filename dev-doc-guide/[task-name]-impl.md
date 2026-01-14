# [task-name]-impl.md

**用途：** 方案设计的任务拆解，跟踪进度

**包含：**
- 按逻辑部分细分的阶段
- 复选框格式的任务
- 状态指示器（✅/🟡/⏳）
- 验收标准


## 任务拆分原则

**核心思想：** 尽可能以端点（Endpoint）为边界，快速提供可测试的接口，便于前后端联调。

**拆分步骤：**
1. **第一层拆分：按对接点划分**
   - HTTP 接口（REST API、GraphQL endpoint）
   - XXL-Job 执行器（定时任务）
   - MQ 监听器（消息队列消费者）
   - RPC 接口（OpenFeign、gRPC 等）
   - 其他可外部对接或测试的任务

2. **第二层拆分：每个对接点内部按层级实现**
   - 数据层（Repository/DAO）
   - 业务层（Service）
   - 接口层（Controller/Listener/Executor）

**优势：**
- ✅ 快速提供可测试的接口
- ✅ 前后端可并行开发联调
- ✅ 每个对接点独立完成，降低依赖
- ✅ 便于增量交付和验证


---

## 任务拆解示例

### 第 1 阶段：基础代码实现 ⏳

**任务 1.1：编写 SQL 脚本**
- [ ] xxx 表新增列 yyy
- [ ] 新建表 zzz
- [ ] 验收：SQL 脚本可执行且符合规范

**任务 1.2：添加数据访问对象**
- [ ] 新建 xxxDO、xxxMapper
- [ ] 添加 MyBatis-Plus 注解
- [ ] 验收：实体类编译通过


### 第 2 阶段：逐个提供端点 ⏳

**端点 2.1：HTTP 接口** `POST /api/xxx`
- [ ] 实现 Service 层
- [ ] 实现 Controller层 xxxController.create()
- [ ] 提供基于springboot的单元测试，打印执行结果
- [ ] 验收：curl -X POST HOST/api/xxx 返回200

**端点 2.2：XXL-Job 定时任务** `xxxJobHandler`
- [ ] 实现 Service 层
- [ ] 实现 JobHandler.handle()
- [ ] 验收：手动触发任务执行成功

**端点 2.3：MQ 监听器** `xxxListener`
- [ ] 实现 Service 层
- [ ] 实现 Listener
- [ ] 提供基于springboot的单元测试，打印执行结果
- [ ] 验收：发送测试消息，监听器正确处理

**端点 2.4：RPC 接口** `XxxFeignClient`
- [ ] 实现 Service 层
- [ ] 实现 FeignClient 接口
- [ ] 提供基于springboot的单元测试，打印执行结果
- [ ] 验收：单元测试通过


### 第 3 阶段：其他团队集成（提供集成指南）⏳

**前端集成**
- [ ] 编写前端集成指南文档
- [ ] 验收：前端按文档完成集成

**业务团队集成**
- [ ] 编写业务接口集成指南
- [ ] 验收：业务团队按文档完成集成  