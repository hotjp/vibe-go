# task_028

## ⚠️ 重要提示（Agent 必读）

**当前位置**: `.long-run-agent/tasks/task_028.md`（任务描述文件）

**工作目录**: 项目根目录（`.long-run-agent` 的同级目录）

**产出物**: 请在项目根目录或适当子目录创建交付物

**这是配置文件**，不是最终产出！

## 描述

T62 Rule L4-Service: CRUD — internal/service/rule.go, 跨模块校验


## 需求 (requirements)

RuleService: CRUD RPC 方法; 创建/更新时: 查 TagValueRepo→TagGroupRepo→TagTypeRepo 获取 apply_subjects→调用 rule.ValidateScope(applySubjects); 创建时校验 TagValueID 存在且 ACTIVE; 创建时校验 entity_type 与 TagType.apply_subjects 兼容(不兼容返回 L2_203); 乐观锁更新+缓存+Outbox



## 验收标准 (acceptance)


- 创建时校验 TagValueID 存在且 ACTIVE; 创建时校验 entity_type 与 TagType.apply_subjects 兼容; 乐观锁更新; CRUD+缓存+Outbox




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