# task_013

## ⚠️ 重要提示（Agent 必读）

**当前位置**: `.long-run-agent/tasks/task_013.md`（任务描述文件）

**工作目录**: 项目根目录（`.long-run-agent` 的同级目录）

**产出物**: 请在项目根目录或适当子目录创建交付物

**这是配置文件**，不是最终产出！

## 描述

T22 TagType L4-Service — internal/service/tag_type.go, 5个RPC方法


## 需求 (requirements)

TagTypeService: CreateTagType/GetTagType/ListTagTypes/UpdateTagType/DisableTagType; 写操作后同步刷新缓存; 创建/更新/禁用后写入 Outbox(TagTypeCreatedV1/TagTypeUpdatedV1); Update 使用乐观锁(version 不匹配返回 L2_204); Create 校验 name 非空+不重复



## 验收标准 (acceptance)


- mock 测试覆盖 5个 RPC 正常路径+校验失败路径; Create 校验 name 非空+不重复; List 支持分页+过滤; 缓存刷新和 Outbox 写入被正确调用




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