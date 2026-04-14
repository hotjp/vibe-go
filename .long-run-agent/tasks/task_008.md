# task_008

## ⚠️ 重要提示（Agent 必读）

**当前位置**: `.long-run-agent/tasks/task_008.md`（任务描述文件）

**工作目录**: 项目根目录（`.long-run-agent` 的同级目录）

**产出物**: 请在项目根目录或适当子目录创建交付物

**这是配置文件**，不是最终产出！

## 描述

T11 数据库迁移脚本（全部表）— migrations/001~007 共 7 组 up/down SQL


## 需求 (requirements)

7张表: tag_type, tag_group, tag_value, tagging, rule, tag_value_changelog, outbox_events; DDL 直接复制 DESIGN.md §4.1; up 顺序执行 + down 逆序执行



## 验收标准 (acceptance)


- up 顺序执行 + down 逆序执行均无错误

- 最终 schema 与 DESIGN.md 一致




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