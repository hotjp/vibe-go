# task_024

## ⚠️ 重要提示（Agent 必读）

**当前位置**: `.long-run-agent/tasks/task_024.md`（任务描述文件）

**工作目录**: 项目根目录（`.long-run-agent` 的同级目录）

**产出物**: 请在项目根目录或适当子目录创建交付物

**这是配置文件**，不是最终产出！

## 描述

T54 Tagging L4-Service: 查询与统计 — internal/service/tagging_query.go


## 需求 (requirements)

GetEntityTags/BatchGetEntityTags/GetTaggingStats; GetEntityTags: 查 TaggingRepo.ListByEntity→对每条 tagging 展开标签链(TagValueRepo→TagGroupRepo→TagTypeRepo); 优先走缓存 miss 时查 DB 并回填; BatchGet 支持多实体; Stats 支持按 entity_type/tag_value_id/时间范围过滤和聚合



## 验收标准 (acceptance)


- 返回完整标签链(Tagging+TagValue+TagGroup+TagType); BatchGet 支持多实体; Stats 支持过滤和聚合




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