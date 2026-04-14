# task_005

## ⚠️ 重要提示（Agent 必读）

**当前位置**: `.long-run-agent/tasks/task_005.md`（任务描述文件）

**工作目录**: 项目根目录（`.long-run-agent` 的同级目录）

**产出物**: 请在项目根目录或适当子目录创建交付物

**这是配置文件**，不是最终产出！

## 描述

T04 共享内核: 错误码 — internal/domain/errors.go


## 需求 (requirements)

NewDomainError(code, message) 返回携带 layer 和 code 的 error; L1_001~L1_010(Storage层), L2_201~L2_204(Domain层), L3_401~L3_402(Authz层), L4_601~L4_603(Service层), L5_801~L5_802(Gateway层); 支持 errors.Is 匹配



## 验收标准 (acceptance)


- 错误码覆盖 DESIGN.md §8 全部条目

- 支持 errors.Is 匹配




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