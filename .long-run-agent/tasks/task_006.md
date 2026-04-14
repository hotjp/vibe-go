# task_006

## ⚠️ 重要提示（Agent 必读）

**当前位置**: `.long-run-agent/tasks/task_006.md`（任务描述文件）

**工作目录**: 项目根目录（`.long-run-agent` 的同级目录）

**产出物**: 请在项目根目录或适当子目录创建交付物

**这是配置文件**，不是最终产出！

## 描述

T05 共享内核: 事件协议 + 状态机框架 + 接口契约 — domain/events.go + domain/statemachine.go + service/interfaces.go


## 需求 (requirements)

domain/events.go: 8种领域事件构造函数(TagTypeCreated/Updated, TagGroupCreated/Updated, TagValueCreated/Updated, TaggingApplied/Revoked); DomainEvent struct 含 event_id(ULID),aggregate_type,aggregate_id,event_type,payload,occurred_at,idempotency_key,version; domain/statemachine.go: 通用状态机(states,transitions,guards,actions),每次转换自动increment_version; service/interfaces.go: TagTypeRepo/TagGroupRepo/TagValueRepo/TaggingRepo/RuleRepo/OutboxWriter/CacheProvider/RuleMatcher/RuleEngineService 全部接口定义



## 验收标准 (acceptance)


- 8种事件构造函数正确生成符合协议的 DomainEvent

- 状态机支持合法转换+拒绝非法转换+自动version递增

- 接口签名完整覆盖所有L1-Storage和Plugin操作




## 交付物 (deliverables)

<!-- 在此填写交付物文件路径 -->



## 设计方案 (design)

<!-- 在此填写架构设计、技术选型、实现思路 -->


## 验证证据（完成前必填）

<!-- 标记完成前，请提供以下证据： -->

- [ ] **实现证明**: 简要说明如何实现
- [ ] **测试验证**: 如何验证功能正常（测试步骤/截图/命令输出）
- [ ] **影响范围**: 是否影响其他功能

### 测试步骤
1. 
2. 
3. 

### 验证结果
<!-- 粘贴验证截图、命令输出或测试结果 -->