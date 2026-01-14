# ClaudeCode 开发文档维护指南

## 修改记录

| Date | Version | Author | Description of Changes |
|------|---------|--------|------------------------|
| 2026-01-13 14:30 | 1.0 | Karl | 初始版本创建 |

## 背景

Claude code 执行过程中，**上下文重置**会丢失一切：
- 实现决策
- 关键文件及其用途
- 任务进度 
- 技术限制  
- 为什么选择某些方法  
  
**重置后，Claude code 必须重新发现一切。**  

## 解决方案：持久化开发文档
  
**三文件结构**，捕获恢复工作所需的一切：

```
docs/active/[task-name]/     # 正在执行的任务
├── [task-name]-plan.md      # 方案设计文档
├── [task-name]-context.md   # 关键决策和进度
└── [task-name]-impl.md      # 任务拆解和跟踪

docs/archived/[task-name]/   # 已经完成的任务，归档
├── [task-name]-plan.md
├── [task-name]-context.md
└── [task-name]-impl.md
```  
  
**这些文件在上下文重置后仍然存在** - Claude 读取它们以立即恢复速度。  
  
---  
  
## 三文件结构

### 1. [task-name]-plan.md

**用途：** 方案设计文档，技术方案的战略规划

**包含：**
- 需求目标
- 当前状态分析
- 实现思路（时序图、流程图）
- 数据库设计（MySQL、Redis、Kafka 等）
- 端点设计（API、RPC、MQ、JOB）
- 补充说明

**示例：**
```markdown
# 帖子管理功能 - 实现方案

## 1. 需求目标

构建一个完整的帖子管理系统，支持帖子的创建、查询、审核和实时更新通知。

**核心功能：**
- 用户可以创建和查询帖子
- 系统定时审核帖子内容
- 帖子状态变更时通知相关服务

## 2. 分析与设计

### 2.1 当前状态

- 现有系统没有帖子管理功能
- 用户模块已完成，可直接使用用户信息
- 已有 Redis 缓存和 Kafka 消息队列基础设施

### 2.2 实现思路

**核心流程：**
1. 用户通过 API 创建帖子 → 保存到数据库 → 返回帖子 ID
2. 用户通过 API 查询帖子 → 先查 Redis 缓存 → 缓存未命中则查数据库
3. 定时任务每小时审核待审核帖子 → 更新审核状态
4. 帖子状态变更时发送 MQ 消息 → 通知其他服务

[Optional] 时序图：
```
用户 -> API: POST /api/posts
API -> Service: createPost()
Service -> DB: save()
Service -> MQ: 发送创建事件
API -> 用户: 返回帖子信息
```

### 2.3 数据库设计

**MySQL 表结构：**
```sql
CREATE TABLE posts (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL COMMENT '用户ID',
    title VARCHAR(200) NOT NULL COMMENT '标题',
    content TEXT NOT NULL COMMENT '内容',
    status VARCHAR(20) DEFAULT 'PENDING' COMMENT '状态：PENDING/APPROVED/REJECTED',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_user_id (user_id),
    INDEX idx_status (status)
) COMMENT='帖子表';
```

**Redis 缓存设计：**
- Key: `post:{id}`
- Value: JSON 格式的帖子信息
- TTL: 1 小时

**Kafka Topic 设计：**
- Topic: `post.events`
- 消息格式: `{"postId": 123, "eventType": "CREATED/UPDATED", "timestamp": 1234567890}`

### 2.4 端点设计

**模块/服务：post-service**

- 【API】 创建帖子：`POST /api/posts`，创建新帖子并返回帖子信息
- 【API】 查询帖子：`GET /api/posts/{id}`，根据 ID 查询帖子详情
- 【JOB】 帖子审核任务：`reviewPostsJob`，每小时执行一次，审核待审核帖子
- 【MQ】  帖子更新监听：`postUpdateListener`，监听 `post.events` Topic，处理帖子更新事件

### 2.5 补充说明

- 帖子内容需要进行敏感词过滤，使用现有的 ContentFilterService
- 审核逻辑初期采用简单规则，后续可接入 AI 审核
- 缓存策略采用 Cache-Aside 模式，写操作时删除缓存
```  
  
### 2. [task-name]-context.md  

**用途：** 恢复工作的关键信息  

**包含：**  
- 进度部分（经常更新！）  
- 已完成 vs 进行中的内容  
- 关键文件及其用途  
- 做出的重要决策  
- 发现的技术限制  
- 相关文件的链接  
- 快速恢复说明  
  
**示例：**
```markdown
# 帖子管理功能 - 上下文

## 进度（2026-01-13）

### ✅ 已完成
**第 1 阶段：创建帖子接口（POST /api/posts）**
- PostRepository.save() 已实现，支持事务
- PostService.createPost() 已实现，包含参数验证和业务逻辑
- PostController.createPost() 已实现，返回 201 状态码
- 验收通过：curl -X POST /api/posts -d '{"title":"test"}' 返回正确

**第 2 阶段：查询帖子接口（GET /api/posts/{id}）**
- PostRepository.findById() 已实现，支持缓存

### 🟡 进行中
**第 2 阶段：查询帖子接口（GET /api/posts/{id}）**
- 正在实现 PostService.getPostById()
- 文件：src/services/PostService.java
- 下一步：添加缓存逻辑，然后实现 Controller 层

### ⚠️ 阻碍
- 需要确认缓存策略：Redis 还是本地缓存？

## 关键文件
**src/repositories/PostRepository.java** ✅
- 帖子数据访问层
- 已实现 save() 和 findById() 方法

**src/services/PostService.java** 🟡
- 帖子业务逻辑层
- createPost() 已完成
- getPostById() 进行中

**src/controllers/PostController.java** 🟡
- 帖子 HTTP 接口层
- createPost() 已完成
- getPost() 待实现

## 技术决策
1. **对接点优先原则**：每个阶段完成一个完整的对接点（endpoint），确保可以独立测试
2. **层级实现顺序**：数据层 → 业务层 → 接口层
3. **验收标准**：每个对接点必须通过 curl 或实际调用测试

## 快速恢复
继续：
1. 阅读此文件了解当前进度
2. 决定缓存策略（询问用户或使用默认 Redis）
3. 完成 PostService.getPostById() 的缓存逻辑
4. 实现 PostController.getPost()
5. 验收：curl -X GET /api/posts/1 返回数据
6. 查看 tasks.md 了解后续阶段（XXL-Job、MQ Listener）
```  
  
**关键：** 每次完成重要工作时都要更新进度部分！

### 3. [task-name]-impl.md

**用途：** 方案设计的任务拆解，跟踪进度

**包含：**
- 按逻辑部分细分的阶段
- 复选框格式的任务
- 状态指示器（✅/🟡/⏳）
- 验收标准

**任务拆分原则：**

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

**示例：**
```markdown
# 帖子管理功能 - 任务拆解

## 第 1 阶段：基础代码实现 ⏳

**任务 1.1：编写 SQL 脚本**
- [ ] 创建 posts 表（包含 id、user_id、title、content、status 等字段）
- [ ] 添加索引（idx_user_id、idx_status）
- [ ] 验收：SQL 脚本可执行且符合规范

**任务 1.2：添加数据访问对象**
- [ ] 新建 PostDO、PostMapper
- [ ] 添加 MyBatis-Plus 注解
- [ ] 验收：实体类编译通过

## 第 2 阶段：逐个提供端点 ⏳

**端点 2.1：创建帖子接口** `POST /api/posts`
- [ ] 实现 PostService.createPost()（业务层）
- [ ] 实现 PostController.createPost()（接口层）
- [ ] 提供基于 SpringBoot 的单元测试，打印执行结果
- [ ] 验收：curl -X POST /api/posts -d '{"title":"test","content":"hello"}' 返回 201

**端点 2.2：查询帖子接口** `GET /api/posts/{id}`
- [ ] 实现 PostService.getPostById()（业务层，包含 Redis 缓存逻辑）
- [ ] 实现 PostController.getPost()（接口层）
- [ ] 提供基于 SpringBoot 的单元测试，打印执行结果
- [ ] 验收：curl -X GET /api/posts/1 返回帖子数据

**端点 2.3：帖子审核定时任务** `reviewPostsJob`
- [ ] 实现 PostReviewService.reviewPendingPosts()（业务层）
- [ ] 实现 ReviewPostsJobHandler.execute()（XXL-Job Handler）
- [ ] 验收：手动触发任务，待审核帖子状态正确更新

**端点 2.4：帖子更新消息监听** `postUpdateListener`
- [ ] 实现 PostEventService.handlePostUpdate()（业务层）
- [ ] 实现 PostUpdateListener.onMessage()（RabbitMQ Listener）
- [ ] 提供基于 SpringBoot 的单元测试，发送测试消息
- [ ] 验收：发送测试消息，监听器正确处理并打印日志

## 第 3 阶段：集成和文档 ⏳

**前端集成**
- [ ] 编写前端集成指南文档（API 接口说明、请求示例、响应格式）
- [ ] 验收：前端按文档完成集成

**运维部署**
- [ ] 编写部署文档（环境变量、依赖服务、启动命令）
- [ ] 验收：测试环境部署成功
```

**关键：** 实际执行时，根据进度更新状态指示器（✅/🟡/⏳）和复选框！  
  