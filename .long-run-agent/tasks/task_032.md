# task_032

## ⚠️ 重要提示（Agent 必读）

**当前位置**: `.long-run-agent/tasks/task_032.md`（任务描述文件）

**工作目录**: 项目根目录（`.long-run-agent` 的同级目录）

**产出物**: 请在项目根目录或适当子目录创建交付物

**这是配置文件**，不是最终产出！

## 描述

T90 Auth-center 客户端 + 拦截器 — internal/authz/client.go + interceptor.go


## 需求 (requirements)

authz/client.go: HTTP 客户端调用 auth-center CheckPermission; authz/interceptor.go: Connect 拦截器(JWT→user_id→权限校验); 权限结果缓存 key: auth:perm:{user_id}:{resource}:{action}; 错误码 L3_401/L3_402; 降级: auth-center 不可用→查缓存→miss→503 Fail Closed; 熔断: 30s 内 10 次失败; httptest mock 测试



## 验收标准 (acceptance)


- JWT 解析正确提取 user_id; CheckPermission 正确调用 auth-center; 降级和熔断逻辑正确; httptest mock 测试




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