# [task-name]-plan.md
  
用途： 实现的战略计划，方案设计
  
包含：  
- 执行摘要  
- 当前状态分析  
- 提议的未来状态
- 风险评估  
- 成功指标

示例：  
```markdown  
# 功能名称 - 实现方案
 
## 1. 需求目标

我们正在构建什么以及为什么  


## 2.分析与设计

### 2.1当前状态

基于需求模板，分析项目当前的状态如何。

### 2.2 实现思路

[Optional] 核心时序图

[Optional] 核心流程图


### 2.3 数据库设计

[Optional] MySQL 表结构设计

[Optional] Postgres 表结构设计

[Optional] Redis k-v结构设计

[Optional] Kafka topic 设计

...


### 2.4 端点设计

[模块名服务名]
- 【API】 用户创建：POST usercreate，[接口说明]
- 【RPC】 用户禁用：POST userban，[接口说明]
- 【MQ】  用户禁用：TOPIC, [Topic说明]
- 【JOB】 用户订阅重置，固定时间corn表达式 触发

 
### 2.5 补充说明

用户授权的流程应该是用标准的 OAuth2 协议，应该采用开源社区的库来实现

···


```