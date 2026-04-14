# task_025

## ⚠️ 重要提示（Agent 必读）

**当前位置**: `.long-run-agent/tasks/task_025.md`（任务描述文件）

**工作目录**: 项目根目录（`.long-run-agent` 的同级目录）

**产出物**: 请在项目根目录或适当子目录创建交付物

**这是配置文件**，不是最终产出！

## 描述

T55 Tagging L4-Service: 批量打标 — internal/service/tagging_batch.go


## 需求 (requirements)

BatchApplyTags(单实体多标签): 部分失败不影响其他,返回每条结果; BatchApplyTagToEntities(多实体单标签): 同上; 复用 T52 ApplyTag 核心逻辑; 返回 success_count/failure_count



## 验收标准 (acceptance)


- BatchApplyTags 部分失败不影响其他; BatchApplyTagToEntities 同上; 返回 success_count/failure_count




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