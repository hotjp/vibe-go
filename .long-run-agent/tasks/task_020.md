# task_020

## ⚠️ 重要提示（Agent 必读）

**当前位置**: `.long-run-agent/tasks/task_020.md`（任务描述文件）

**工作目录**: 项目根目录（`.long-run-agent` 的同级目录）

**产出物**: 请在项目根目录或适当子目录创建交付物

**这是配置文件**，不是最终产出！

## 描述

T50 Tagging L2-Domain — internal/domain/tagging.go


## 需求 (requirements)

domain.Tagging struct: ID,EntityType,EntityID,TagValueID,Source,OperatorID,Confidence,Status,Remark,Version,CreatedAt,UpdatedAt; NewTagging()(默认 status=ACTIVE,手动 confidence=1.0); Revoke()(ACTIVE→REVOKED); IsIdempotentWith(other) bool(幂等判断: 同 entity_type+entity_id+tag_value_id+status=ACTIVE); 幂等: 同(EntityType,EntityID,TagValueID)只能一条 ACTIVE; 撤销不物理删除



## 验收标准 (acceptance)


- NewTagging 默认值正确; Revoke 仅 ACTIVE→REVOKED; IsIdempotentWith 正确判断幂等




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