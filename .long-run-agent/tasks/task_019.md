# task_019

## ⚠️ 重要提示（Agent 必读）

**当前位置**: `.long-run-agent/tasks/task_019.md`（任务描述文件）

**工作目录**: 项目根目录（`.long-run-agent` 的同级目录）

**产出物**: 请在项目根目录或适当子目录创建交付物

**这是配置文件**，不是最终产出！

## 描述

T42 TagValue L4-Service — internal/service/tag_value.go, 双路径更新


## 需求 (requirements)

TagValueService: CRUD RPC + GetValueChangeLog; 创建时校验 TagGroupID 存在且 ACTIVE(调 TagGroupRepo); Update 路径: 根据 is_semantic_change 参数走 UpdateMetadata()或 SemanticChange(); SemanticChange 路径额外写入 changelog+发布 TagValueUpdatedV1 事件; 乐观锁+缓存+Outbox



## 验收标准 (acceptance)


- 创建校验 TagGroupID 存在且 ACTIVE; Update 正确区分两条路径; SemanticChange 写 changelog+发 Outbox 事件; GetValueChangeLog 查询正确




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