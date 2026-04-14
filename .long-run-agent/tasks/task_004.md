# task_004

## ⚠️ 重要提示（Agent 必读）

**当前位置**: `.long-run-agent/tasks/task_004.md`（任务描述文件）

**工作目录**: 项目根目录（`.long-run-agent` 的同级目录）

**产出物**: 请在项目根目录或适当子目录创建交付物

**这是配置文件**，不是最终产出！

## 描述

T03 共享内核: 通用类型 + ULID — domain/types.go 枚举 + domain/ulid.go


## 需求 (requirements)

domain/types.go: Status(0=Unspecified,1=Active,2=Disabled), TagSource(0=Unspecified,1=Manual,2=AutoBuiltin,3=AutoExternal), TagStatus(0=Unspecified,1=Active,2=Revoked), EngineType(0=Unspecified,1=Builtin,2=External) + String()方法; domain/ulid.go: NewID() string; 零外部依赖(ulid除外)



## 验收标准 (acceptance)


- go test ./internal/domain/... 通过

- 枚举值与 DESIGN.md 一致




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