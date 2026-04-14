# task_031

## ⚠️ 重要提示（Agent 必读）

**当前位置**: `.long-run-agent/tasks/task_031.md`（任务描述文件）

**工作目录**: 项目根目录（`.long-run-agent` 的同级目录）

**产出物**: 请在项目根目录或适当子目录创建交付物

**这是配置文件**，不是最终产出！

## 描述

T80 AutoTag L4-Service — internal/service/auto_tag.go, 跨模块编排双通路


## 需求 (requirements)

AutoTag(entity_type,entity_id,content,context)→1.RuleRepo.List(filter:entity_type,status=ACTIVE)→2.errgroup 并发执行规则匹配(BUILTIN→KeywordMatcher,EXTERNAL→ExternalClient)→3.收集匹配结果按Priority排序取最高→4.调用 T52.ApplyTag()(source=AUTO_BUILTIN/AUTO_EXTERNAL)→5.返回匹配结果; errgroup 限制并发数; 单条规则匹配失败不阻塞其他; 错误码 L4_602/L4_603



## 验收标准 (acceptance)


- mock 测试覆盖:无匹配/单匹配/多匹配按优先级/外部超时不阻塞/最终调用 ApplyTag




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