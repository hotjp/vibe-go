# task_010

## ⚠️ 重要提示（Agent 必读）

**当前位置**: `.long-run-agent/tasks/task_010.md`（任务描述文件）

**工作目录**: 项目根目录（`.long-run-agent` 的同级目录）

**产出物**: 请在项目根目录或适当子目录创建交付物

**这是配置文件**，不是最终产出！

## 描述

T13 Outbox 系统 — storage/postgres/outbox_writer.go + outbox_poller.go + storage/redis/event_publisher.go


## 需求 (requirements)

outbox_writer.go: 事务内写入 outbox_events,实现 service.OutboxWriter 接口; outbox_poller.go: 后台轮询 PENDING 事件; event_publisher.go: 发布到 Redis Stream tag-sense:events; 状态: 0=PENDING,1=PUBLISHED,2=FAILED,失败重试



## 验收标准 (acceptance)


- 写入+轮询+发布到 Redis Stream 完整链路

- 失败重试+状态流转正确

- go test ./internal/storage/... -run TestOutbox




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