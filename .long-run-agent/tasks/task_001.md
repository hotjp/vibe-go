# task_001

**状态**: completed
**认领者**: Claude Code Agent
**认领时间**: 2026-04-14T18:19+08:00

## ⚠️ 重要提示（Agent 必读）

**当前位置**: `.long-run-agent/tasks/task_001.md`（任务描述文件）

**工作目录**: 项目根目录（`.long-run-agent` 的同级目录）

**产出物**: 请在项目根目录或适当子目录创建交付物

**这是配置文件**，不是最终产出！

## 描述

T00 项目初始化 — go mod init、完整目录结构、.gitignore、Makefile


## 需求 (requirements)

go mod init github.com/{org}/tag-sense; 创建完整目录结构(cmd/server, internal/gateway, internal/authz, internal/service, internal/domain, internal/storage, plugins, api/tagsense/v1); .gitignore; Makefile(build/test/lint targets); Go 1.23+



## 验收标准 (acceptance)


- go build ./... 无错误

- make build 正常执行




## 交付物 (deliverables)

<!-- 在此填写交付物文件路径 -->
- go.mod (Go 1.23+ module definition)
- .gitignore (Go 项目标准 + 敏感文件)
- Makefile (build/test/lint/clean/fmt/tidy targets)
- cmd/server/main.go (应用入口)
- internal/gateway/gateway.go (L5-Gateway 层)
- internal/authz/authz.go (L3-Authz 层)
- internal/service/service.go (L4-Service 层)
- internal/domain/domain.go (L2-Domain 层，零外部依赖)
- internal/storage/storage.go (L1-Storage 层)
- plugins/plugins.go (插件层接口定义)
- api/tagsense/v1/tagsense.go (API 定义占位)



## 设计方案 (design)

<!-- 在此填写架构设计、技术选型、实现思路 -->
### 架构分层（5 层核心 + N 插件）
```
依赖方向：L5-Gateway → L3-Authz → L4-Service → L2-Domain → L1-Storage
```

### 分层职责
| 层 | 职责 | 关键约束 |
|---|---|---|
| L5-Gateway | TLS 终止、协议适配、中间件、请求路由 | JWT 仅解密不验证，调用 L3 |
| L3-Authz | 权限检查、Rate Limiting、身份验证 | Fail Fast 原则 |
| L4-Service | 输入校验、事务边界、工作流触发、领域协调 | 通过 interface 依赖插件 |
| L2-Domain | 领域实体、状态机、事件收集 (Outbox) | 纯 Go struct，零外部依赖 |
| L1-Storage | Ent 实现、事务管理、Outbox 轮询 | Outbox 同库同事务 |

### 技术选型
- Go 1.23+ (当前环境 Go 1.26.1)
- Module: github.com/vibe-go/tag-sense
- 后续将引入：connect-go, ent, pgx, go-redis, koanf, slog, opentelemetry


## 验证证据（完成前必填）

<!-- 标记完成前，请提供以下证据： -->

- [x] **实现证明**: 创建完整 Go 项目结构，包含 5 层架构骨架代码
- [x] **测试验证**: go build ./... 无错误通过
- [x] **影响范围**: 无，这是新项目初始化

### 测试步骤
1. 运行 `go build ./...` 验证所有包可编译
2. 运行 `make build` 验证 Makefile 构建目标
3. 运行 `git status` 确认所有文件已追踪

### 验证结果
```
$ go build ./...
(no output - success)

$ make build
Building server...
(构建成功，生成 bin/server 二进制文件)
```