# task_012

## ⚠️ 重要提示（Agent 必读）

**当前位置**: `.long-run-agent/tasks/task_012.md`（任务描述文件）

**工作目录**: 项目根目录（`.long-run-agent` 的同级目录）

**产出物**: 请在项目根目录或适当子目录创建交付物

**这是配置文件**，不是最终产出！

## 描述

T21 TagType L1-Storage — Ent schema + Repository, 实现 TagTypeRepo 接口


## 需求 (requirements)

Ent schema: internal/storage/postgres/ent/schema/tag_type.go(含 GIN 索引 on apply_subjects,apply_channels); Repository: internal/storage/postgres/tag_type_repo.go; 实现 service.TagTypeRepo 接口(Create/GetByID/GetByName/List/Update); Ent↔Domain 模型转换在 repo 内部



## 验收标准 (acceptance)


- testcontainers 集成测试覆盖 CRUD+分页+按 status/subject/channel 过滤; 唯一约束冲突返回 L1_002; 外键约束违反返回 L1_003




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