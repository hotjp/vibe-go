# task_017

## ⚠️ 重要提示（Agent 必读）

**当前位置**: `.long-run-agent/tasks/task_017.md`（任务描述文件）

**工作目录**: 项目根目录（`.long-run-agent` 的同级目录）

**产出物**: 请在项目根目录或适当子目录创建交付物

**这是配置文件**，不是最终产出！

## 描述

T40 TagValue L2-Domain — internal/domain/tag_value.go, 含版本控制


## 需求 (requirements)

domain.TagValue struct: ID,TagGroupID,Name,Description,SortOrder,Status,Version,CreatedAt,UpdatedAt; NewTagValue(); UpdateMetadata()(名称/描述/排序变更,不递增version); SemanticChange(reason,operatorID)(语义变更,递增version+产出ChangeLogEntry); Disable(); 区分两种更新路径是核心复杂度



## 验收标准 (acceptance)


- go test ./internal/domain/... -run TestTagValue; NewTagValue/UpdateMetadata 不递增version/SemanticChange 递增version并产出changelog/Disable




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