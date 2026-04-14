# task_014

## ⚠️ 重要提示（Agent 必读）

**当前位置**: `.long-run-agent/tasks/task_014.md`（任务描述文件）

**工作目录**: 项目根目录（`.long-run-agent` 的同级目录）

**产出物**: 请在项目根目录或适当子目录创建交付物

**这是配置文件**，不是最终产出！

## 描述

T30 TagGroup L2-Domain — internal/domain/tag_group.go


## 需求 (requirements)

domain.TagGroup struct: ID,TagTypeID,Name,Description,Status,Version,CreatedAt,UpdatedAt; NewTagGroup() 创建; Disable(); Name 在同一 TagTypeID 下唯一; 适用范围继承自 TagType(不重复存储); 使用 T05 通用状态机框架



## 验收标准 (acceptance)


- go test ./internal/domain/... -run TestTagGroup; 同 T20 模式




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