# task_022

## ⚠️ 重要提示（Agent 必读）

**当前位置**: `.long-run-agent/tasks/task_022.md`（任务描述文件）

**工作目录**: 项目根目录（`.long-run-agent` 的同级目录）

**产出物**: 请在项目根目录或适当子目录创建交付物

**这是配置文件**，不是最终产出！

## 描述

T52 Tagging L4-Service: ApplyTag — internal/service/tagging_apply.go, 手动打标流程


## 需求 (requirements)

跨模块编排核心: ApplyTag(entity_type,entity_id,tag_value_id)→1.TagValueRepo.GetByID(校验存在且ACTIVE)→2.TagGroupRepo.GetByID(获取tag_type_id)→3.TagTypeRepo.GetByID(获取apply_subjects)→4.校验entity_type∈apply_subjects(否则L2_203)→5.FindActive幂等判断→6.NewTagging+Create(事务内)→7.OutboxWriter.Write(TaggingAppliedV1,同事务)→8.Cache.Delete; 不调其他Service直接用Repo; 步骤6-7同一事务



## 验收标准 (acceptance)


- mock 测试覆盖:正常打标/TagValue不存在/TagValue已禁用/entity_type不在apply_subjects/幂等返回已有记录; 事务内Create+Outbox Write被一起调用




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