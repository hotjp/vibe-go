# [Demo] Tag Sense - 标签系统后端设计文档

> **本文档是框架的示例业务设计**，用于验证 vibe-go 脚手架的可用性。不属于框架本身。

## 一、概述

### 1.1 系统定位

Tag Sense 是一个独立的标签管理后端服务，为新媒体舆情场景提供标签全生命周期管理与打标能力。本系统作为标签权威源（Source of Truth），负责标签定义、维护、打标、查询，不管理舆情数据本身。

### 1.2 核心能力

- 标签类型/标签组/标签值的三级管理
- 对任意外部实体的通用打标（舆情、艺人、品牌等）
- 内置规则引擎 + 外部规则服务双通路自动打标
- 标签值版本管理（语义变更追踪，非名称变更）
- 通过 Connect 协议对外提供 gRPC/HTTP 双模 API

### 1.3 设计决策

| 决策项 | 结论 |
|---|---|
| 技术栈 | Go + PostgreSQL + Redis |
| API 协议 | Connect (gRPC + HTTP 双模) |
| 权限 | 对接外部 auth-center (OpenFGA)，本项目不内置用户/角色管理 |
| 标签结构 | 三级（类型→组→值），类型层定义适用范围，下层继承 |
| 版本控制 | 名称修改原地更新，语义变更生成新版本 |
| 打标对象 | 通用关联（entity_type + entity_id），不管理外部实体数据 |
| 规则引擎 | 内置关键词匹配 + 外部规则服务接口，双通路 |
| 分库分表 | 暂不实施，PostgreSQL 分区表预留 |

### 1.4 术语表

| 术语 | 说明 |
|---|---|
| TagType | 标签类型，如"情感标签"、"风险等级标签" |
| TagGroup | 标签组，隶属于 TagType，如"通用"、"艺人专用" |
| TagValue | 标签值，隶属于 TagGroup，如"正面"、"高风险" |
| Tagging / 打标 | 将 TagValue 关联到外部实体的操作 |
| Entity | 外部实体，由 entity_type + entity_id 标识 |
| Rule | 自动打标规则，可内置匹配或调用外部规则服务 |

---

## 二、架构设计

### 2.1 分层架构（遵循 CLAUDE.md 框架）

```
L5-Gateway (Connect handler, 中间件, 路由)
    ↓
L3-Authz (对接外部 auth-center, 请求准入)
    ↓
L4-Service (业务编排: 打标流程, 规则调度, 输入校验)
    ↓
L2-Domain (领域核心: 标签实体, 状态机, 领域事件)
    ↓
L1-Storage (PostgreSQL/Redis 数据访问, Outbox)
```

### 2.2 插件层

```
P1-RuleEngine (规则引擎插件)
  ├── 内置规则匹配器 (关键词/条件匹配)
  └── 外部规则服务适配器 (HTTP/gRPC 调用)
```

### 2.3 外部依赖

```
┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│  业务系统     │────►│  Tag Sense   │◄────│  auth-center │
│  (调用打标API)│     │  (本系统)     │     │  (OpenFGA)   │
└──────────────┘     └──────┬───────┘     └──────────────┘
                           │
                     ┌─────┴─────┐
                     │           │
                ┌────▼───┐  ┌───▼────┐
                │PostgreSQL│  │ Redis  │
                └────────┘  └────────┘
```

### 2.4 项目结构

```
tag-sense/
├── cmd/
│   └── server/
│       └── main.go              # 入口, 依赖注入
├── internal/
│   ├── gateway/                 # L5: Connect handler, 中间件
│   │   ├── handler.go
│   │   ├── middleware.go
│   │   └── routes.go
│   ├── authz/                   # L3: 对接 auth-center
│   │   ├── client.go
│   │   └── interceptor.go
│   ├── service/                 # L4: 业务编排
│   │   ├── interfaces.go        # 插件接口定义
│   │   ├── tag_type.go
│   │   ├── tag_group.go
│   │   ├── tag_value.go
│   │   ├── tagging.go
│   │   └── rule.go
│   ├── domain/                  # L2: 领域核心
│   │   ├── tag_type.go          # 实体定义
│   │   ├── tag_group.go
│   │   ├── tag_value.go
│   │   ├── tagging.go
│   │   ├── rule.go
│   │   ├── events.go            # 领域事件
│   │   └── errors.go            # 领域错误码
│   └── storage/                 # L1: 数据持久化
│       ├── postgres/
│       │   ├── tag_type.go
│       │   ├── tag_group.go
│       │   ├── tag_value.go
│       │   ├── tagging.go
│       │   ├── rule.go
│       │   ├── outbox.go
│       │   └── migrations/
│       └── redis/
│           ├── cache.go
│           └── event_publisher.go
├── plugins/
│   └── ruleengine/              # P1: 规则引擎插件
│       ├── builtin/             # 内置规则匹配
│       │   ├── keyword.go
│       │   └── condition.go
│       └── external/            # 外部规则服务适配
│           └── client.go
├── api/
│   └── tagsense/v1/
│       └── tagsense.proto       # Protobuf 定义
├── buf.yaml
├── buf.gen.yaml
├── go.mod
├── go.sum
├── Makefile
├── scripts/
│   └── check_architecture.sh
├── docs/
│   ├── DESIGN.md                # 本文档
│   ├── DESIGN.pdf               # 原始设计
│   └── architecture.md          # 架构框架
├── CLAUDE.md
├── .gitignore
└── .claude/
```

---

## 三、领域模型

### 3.1 核心实体关系

```
TagType (1) ──► (N) TagGroup (1) ──► (N) TagValue
                                            │
                                            ▼
Tagging ──► TagValue (多对一)
   │
   ▼
Entity (entity_type + entity_id, 逻辑概念, 无实体表)
```

### 3.2 TagType（标签类型）

标签类型的定义，全局唯一，定义适用范围。

```go
type TagType struct {
    ID            string    // ULID
    Name          string    // 类型名称, 全局唯一, 如 "情感标签"
    Description   string    // 类型说明
    ApplySubjects []string  // 适用主体 ["artist", "influencer", "brand", "product"]
    ApplyChannels []string  // 适用渠道 ["weibo", "douyin", "douban", "kuaishou", "weixin_video"]
    Status        Status    // ACTIVE | DISABLED
    Version       int       // 乐观锁版本号
    CreatedAt     time.Time
    UpdatedAt     time.Time
}
```

**业务规则**：
- `Name` 全局唯一，不允许重复
- `ApplySubjects` 和 `ApplyChannels` 存储为 PostgreSQL 数组类型
- 禁用后不可用于新打标，保留历史数据
- 不提供硬删操作，统一走禁用

### 3.3 TagGroup（标签组）

隶属于某个 TagType，按场景细分。

```go
type TagGroup struct {
    ID          string    // ULID
    TagTypeID   string    // 所属标签类型 ID
    Name        string    // 组名称, 同一 TagType 内唯一
    Description string    // 组说明
    Status      Status    // ACTIVE | DISABLED
    SortOrder   int       // 展示排序
    Version     int       // 乐观锁版本号
    CreatedAt   time.Time
    UpdatedAt   time.Time
}
```

**业务规则**：
- `Name` 在同一 `TagTypeID` 下唯一（组内去重）
- 适用范围继承自 TagType，不重复存储
- 不提供硬删操作，统一走禁用

### 3.4 TagValue（标签值）

隶属于某个 TagGroup，是最小打标单位。

```go
type TagValue struct {
    ID          string    // ULID
    TagGroupID  string    // 所属标签组 ID
    Name        string    // 值名称, 如 "正面", "高风险"
    Description string    // 值说明/打标依据
    Status      Status    // ACTIVE | DISABLED
    SortOrder   int       // 展示排序
    Version     int       // 语义版本号, 仅语义变更时递增
    CreatedAt   time.Time
    UpdatedAt   time.Time
}
```

**版本控制规则**：
- **修改名称/描述/排序**：原地更新，不递增 Version
- **语义变更**（如值含义发生本质变化）：递增 Version，写入变更日志
- tag_relation 关联 TagValue.ID，展示时始终显示当前名称
- Version 字段用于审计追溯，不创建新行

**去重规则**：
- `Name` 在同一 `TagGroupID` 下唯一

### 3.5 Tagging（打标记录）

将 TagValue 关联到外部实体。

```go
type Tagging struct {
    ID          string    // ULID
    EntityType  string    // 实体类型, 如 "sentiment", "artist", "brand"
    EntityID    string    // 实体 ID (外部系统标识)
    TagValueID  string    // 关联的标签值 ID
    Source      TagSource // 打标来源: MANUAL | AUTO_BUILTIN | AUTO_EXTERNAL
    OperatorID  string    // 操作人 ID (手动打标为用户, 自动打标为系统/规则ID)
    Confidence  float64   // 置信度 (自动打标时填写, 手动为 1.0)
    Status      TagStatus // ACTIVE | REVOKED
    Remark      string    // 备注
    Version     int       // 乐观锁版本号
    CreatedAt   time.Time
    UpdatedAt   time.Time
}
```

**业务规则**：
- 同一 (EntityType, EntityID, TagValueID) 只能有一条 ACTIVE 记录
- **幂等行为**：重复 ApplyTag 同一 entity + tag_value → 返回已有 ACTIVE 记录，不报错
- 撤销打标不物理删除，将 Status 改为 REVOKED
- 撤销后可再次打标同一标签（创建新 ACTIVE 记录）
- 打标时校验 TagValue 所属 TagType 的适用范围

### 3.6 Rule（打标规则）

自动打标的规则定义。

```go
type Rule struct {
    ID           string      // ULID
    Name         string      // 规则名称
    Description  string      // 规则说明
    TagValueID   string      // 匹配成功后打标的标签值 ID
    EngineType   EngineType  // BUILTIN | EXTERNAL
    MatchConfig  MatchConfig // 匹配配置 (JSON, 按引擎类型不同)
    Priority     int         // 优先级, 数值越大越高
    EntityType   string      // 适用实体类型
    Status       Status      // ACTIVE | DISABLED
    Version      int         // 乐观锁版本号
    CreatedAt    time.Time
    UpdatedAt    time.Time
}

type MatchConfig struct {
    // BUILTIN 引擎
    Keywords    []string      `json:"keywords,omitempty"`     // 关键词列表
    MatchMode   string        `json:"match_mode,omitempty"`   // ANY | ALL
    // EXTERNAL 引擎
    Endpoint    string        `json:"endpoint,omitempty"`     // 外部规则服务地址
    Method      string        `json:"method,omitempty"`       // HTTP 方法
    Headers     map[string]string `json:"headers,omitempty"`  // 请求头
    // 通用
    Timeout     int           `json:"timeout,omitempty"`      // 超时(毫秒)
}
```

**业务规则**：
- 多条规则匹配到同一 Entity 时，按 Priority 取最高
- BUILTIN 类型使用内置关键词/条件匹配
- EXTERNAL 类型通过 HTTP 调用外部规则服务
- 规则禁用后不参与匹配
- **创建/更新时校验**：Rule.entity_type 必须与 TagValue 所属 TagType 的 apply_subjects 兼容（如 TagType 只适用于 artist，Rule 却配 entity_type=brand → 拒绝）

---

## 四、数据库设计

### 4.1 PostgreSQL 表结构

#### 4.1.1 tag_type（标签类型）

```sql
CREATE TABLE tag_type (
    id              VARCHAR(26) PRIMARY KEY,          -- ULID
    name            VARCHAR(50) NOT NULL UNIQUE,      -- 类型名称, 全局唯一
    description     VARCHAR(255),                     -- 类型说明
    apply_subjects  TEXT[] NOT NULL DEFAULT '{}',     -- 适用主体数组
    apply_channels  TEXT[] NOT NULL DEFAULT '{}',     -- 适用渠道数组
    status          SMALLINT NOT NULL DEFAULT 1,      -- 1=ACTIVE, 0=DISABLED
    version         INT NOT NULL DEFAULT 1,           -- 乐观锁
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_tag_type_status ON tag_type(status);
CREATE INDEX idx_tag_type_subjects ON tag_type USING GIN(apply_subjects);
CREATE INDEX idx_tag_type_channels ON tag_type USING GIN(apply_channels);
```

#### 4.1.2 tag_group（标签组）

```sql
CREATE TABLE tag_group (
    id              VARCHAR(26) PRIMARY KEY,
    tag_type_id     VARCHAR(26) NOT NULL REFERENCES tag_type(id),
    name            VARCHAR(50) NOT NULL,             -- 同 tag_type_id 下唯一
    description     VARCHAR(255),
    status          SMALLINT NOT NULL DEFAULT 1,
    sort_order      INT NOT NULL DEFAULT 0,
    version         INT NOT NULL DEFAULT 1,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_tag_group_type_name UNIQUE (tag_type_id, name)
);

CREATE INDEX idx_tag_group_type_status ON tag_group(tag_type_id, status);
```

#### 4.1.3 tag_value（标签值）

```sql
CREATE TABLE tag_value (
    id              VARCHAR(26) PRIMARY KEY,
    tag_group_id    VARCHAR(26) NOT NULL REFERENCES tag_group(id),
    name            VARCHAR(50) NOT NULL,             -- 同 tag_group_id 下唯一
    description     VARCHAR(255),
    status          SMALLINT NOT NULL DEFAULT 1,
    sort_order      INT NOT NULL DEFAULT 0,
    version         INT NOT NULL DEFAULT 1,           -- 语义版本号
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    CONSTRAINT uq_tag_value_group_name UNIQUE (tag_group_id, name)
);

CREATE INDEX idx_tag_value_group_status ON tag_value(tag_group_id, status, sort_order);
```

#### 4.1.4 tagging（打标记录）

```sql
CREATE TABLE tagging (
    id              VARCHAR(26) PRIMARY KEY,
    entity_type     VARCHAR(50) NOT NULL,             -- 实体类型
    entity_id       VARCHAR(100) NOT NULL,            -- 实体 ID
    tag_value_id    VARCHAR(26) NOT NULL REFERENCES tag_value(id),
    source          SMALLINT NOT NULL,                -- 1=MANUAL, 2=AUTO_BUILTIN, 3=AUTO_EXTERNAL
    operator_id     VARCHAR(50) NOT NULL,             -- 操作人
    confidence      DECIMAL(5,4) NOT NULL DEFAULT 1.0,-- 置信度
    status          SMALLINT NOT NULL DEFAULT 1,      -- 1=ACTIVE, 2=REVOKED
    remark          VARCHAR(500),
    version         INT NOT NULL DEFAULT 1,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),

    -- 同一实体+标签值只允许一条 ACTIVE 记录
    CONSTRAINT uq_tagging_entity_tag_active UNIQUE (entity_type, entity_id, tag_value_id)
        WHERE status = 1
);

CREATE INDEX idx_tagging_entity ON tagging(entity_type, entity_id);
CREATE INDEX idx_tagging_entity_active ON tagging(entity_type, entity_id, status);
CREATE INDEX idx_tagging_tag_value ON tagging(tag_value_id);
CREATE INDEX idx_tagging_source ON tagging(source, created_at);
CREATE INDEX idx_tagging_created ON tagging(created_at);
```

#### 4.1.5 rule（打标规则）

```sql
CREATE TABLE rule (
    id              VARCHAR(26) PRIMARY KEY,
    name            VARCHAR(100) NOT NULL UNIQUE,
    description     VARCHAR(500),
    tag_value_id    VARCHAR(26) NOT NULL REFERENCES tag_value(id),
    engine_type     SMALLINT NOT NULL,                -- 1=BUILTIN, 2=EXTERNAL
    match_config    JSONB NOT NULL,                   -- 匹配配置
    priority        INT NOT NULL DEFAULT 0,
    entity_type     VARCHAR(50) NOT NULL,             -- 适用实体类型
    status          SMALLINT NOT NULL DEFAULT 1,
    version         INT NOT NULL DEFAULT 1,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_rule_tag_value ON rule(tag_value_id);
CREATE INDEX idx_rule_engine_status ON rule(engine_type, status, entity_type);
```

#### 4.1.6 tag_value_changelog（标签值变更日志）

记录标签值的语义变更历史，用于审计追溯。

```sql
CREATE TABLE tag_value_changelog (
    id              VARCHAR(26) PRIMARY KEY,
    tag_value_id    VARCHAR(26) NOT NULL REFERENCES tag_value(id),
    old_version     INT NOT NULL,
    new_version     INT NOT NULL,
    change_type     VARCHAR(20) NOT NULL,             -- NAME_CHANGE | DESC_CHANGE | SEMANTIC_CHANGE
    old_value       JSONB,                            -- 变更前快照
    new_value       JSONB,                            -- 变更后快照
    operator_id     VARCHAR(50) NOT NULL,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_changelog_tag_value ON tag_value_changelog(tag_value_id, created_at DESC);
```

#### 4.1.7 outbox_events（Outbox 事件表）

领域事件通过 Outbox 模式发布，确保与业务事务的一致性。

```sql
CREATE TABLE outbox_events (
    id              VARCHAR(26) PRIMARY KEY,
    aggregate_type  VARCHAR(50) NOT NULL,
    aggregate_id    VARCHAR(26) NOT NULL,
    event_type      VARCHAR(100) NOT NULL,            -- 如 TagCreatedV1
    payload         JSONB NOT NULL,
    occurred_at     TIMESTAMPTZ NOT NULL,
    idempotency_key VARCHAR(200) NOT NULL UNIQUE,
    status          SMALLINT NOT NULL DEFAULT 0,      -- 0=PENDING, 1=PUBLISHED, 2=FAILED
    retry_count     INT NOT NULL DEFAULT 0,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE INDEX idx_outbox_status ON outbox_events(status, created_at);
```

### 4.2 Redis 使用设计

#### 4.2.1 缓存策略

| Key 模式 | 数据内容 | 过期时间 | 更新策略 |
|---|---|---|---|
| `tag:type:{id}` | TagType 实体 | 30min | 更新即刷新 |
| `tag:type:all` | 全部 TagType 列表 | 10min | 更新即刷新 |
| `tag:group:{id}` | TagGroup 实体 | 30min | 更新即刷新 |
| `tag:group:type:{type_id}` | 某类型下所有 TagGroup | 10min | 更新即刷新 |
| `tag:value:{id}` | TagValue 实体 | 30min | 更新即刷新 |
| `tag:value:group:{group_id}` | 某组下所有 TagValue | 10min | 更新即刷新 |
| `tag:entity:{entity_type}:{entity_id}` | 某实体的所有打标 | 5min | 打标时刷新 |

#### 4.2.2 事件流

Outbox 轮询器将 PENDING 事件发布到 Redis Stream：
- Stream 名称：`tag-sense:events`
- Consumer Group：各业务系统按需创建

---

## 五、API 设计

### 5.1 Protobuf Service 定义

```protobuf
syntax = "proto3";
package tagsense.v1;

import "google/protobuf/field_mask.proto";

service TagSenseService {
  // === 标签类型 ===
  rpc CreateTagType(CreateTagTypeRequest) returns (TagTypeResponse);
  rpc GetTagType(GetTagTypeRequest) returns (TagTypeResponse);
  rpc ListTagTypes(ListTagTypesRequest) returns (ListTagTypesResponse);
  rpc UpdateTagType(UpdateTagTypeRequest) returns (TagTypeResponse);
  rpc DisableTagType(DisableTagTypeRequest) returns (TagTypeResponse);
  // 注意：无 Delete RPC，禁用即为"删除"。原因：硬删会破坏 FK 引用和历史打标数据。

  // === 标签组 ===
  rpc CreateTagGroup(CreateTagGroupRequest) returns (TagGroupResponse);
  rpc GetTagGroup(GetTagGroupRequest) returns (TagGroupResponse);
  rpc ListTagGroups(ListTagGroupsRequest) returns (ListTagGroupsResponse);
  rpc UpdateTagGroup(UpdateTagGroupRequest) returns (TagGroupResponse);
  rpc DisableTagGroup(DisableTagGroupRequest) returns (TagGroupResponse);

  // === 标签值 ===
  rpc CreateTagValue(CreateTagValueRequest) returns (TagValueResponse);
  rpc GetTagValue(GetTagValueRequest) returns (TagValueResponse);
  rpc ListTagValues(ListTagValuesRequest) returns (ListTagValuesResponse);
  rpc UpdateTagValue(UpdateTagValueRequest) returns (TagValueResponse);
  rpc DisableTagValue(DisableTagValueRequest) returns (TagValueResponse);

  // === 打标 ===
  rpc ApplyTag(ApplyTagRequest) returns (TaggingResponse);
  rpc BatchApplyTags(BatchApplyTagsRequest) returns (BatchApplyTagsResponse);         // 单实体多标签
  rpc BatchApplyTagToEntities(BatchApplyTagToEntitiesRequest) returns (BatchApplyTagsResponse);  // 多实体单标签
  rpc RevokeTag(RevokeTagRequest) returns (.google.protobuf.Empty);
  rpc GetEntityTags(GetEntityTagsRequest) returns (GetEntityTagsResponse);
  rpc BatchGetEntityTags(BatchGetEntityTagsRequest) returns (BatchGetEntityTagsResponse);

  // === 自动打标 (规则) ===
  rpc AutoTag(AutoTagRequest) returns (AutoTagResponse);
  rpc CreateRule(CreateRuleRequest) returns (RuleResponse);
  rpc GetRule(GetRuleRequest) returns (RuleResponse);
  rpc ListRules(ListRulesRequest) returns (ListRulesResponse);
  rpc UpdateRule(UpdateRuleRequest) returns (RuleResponse);
  rpc DisableRule(DisableRuleRequest) returns (RuleResponse);

  // === 查询统计 ===
  rpc GetTaggingStats(GetTaggingStatsRequest) returns (GetTaggingStatsResponse);
  rpc GetValueChangeLog(GetValueChangeLogRequest) returns (GetValueChangeLogResponse);
}
```

### 5.2 核心 Message 定义

```protobuf
// --- 标签类型 ---
message TagTypePB {
  string id = 1;
  string name = 2;
  string description = 3;
  repeated string apply_subjects = 4;
  repeated string apply_channels = 5;
  int32 status = 6;
  int32 version = 7;
  string created_at = 8;
  string updated_at = 9;
}

message CreateTagTypeRequest {
  string name = 1;
  string description = 2;
  repeated string apply_subjects = 3;
  repeated string apply_channels = 4;
}

message UpdateTagTypeRequest {
  string id = 1;
  optional string name = 2;
  optional string description = 3;
  repeated string apply_subjects = 4;
  repeated string apply_channels = 5;
  int32 version = 6;  // 乐观锁
  google.protobuf.FieldMask update_mask = 7;
}

message ListTagTypesRequest {
  int32 page_size = 1;
  string page_token = 2;
  optional int32 status = 3;
  optional string subject = 4;   // 按适用主体过滤
  optional string channel = 5;   // 按适用渠道过滤
}

message ListTagTypesResponse {
  repeated TagTypePB items = 1;
  string next_page_token = 2;
  int32 total = 3;
}

// --- 标签组 ---
message TagGroupPB {
  string id = 1;
  string tag_type_id = 2;
  string name = 3;
  string description = 4;
  int32 status = 5;
  int32 sort_order = 6;
  int32 version = 7;
  string created_at = 8;
  string updated_at = 9;
}

message CreateTagGroupRequest {
  string tag_type_id = 1;
  string name = 2;
  string description = 3;
  int32 sort_order = 4;
}

message ListTagGroupsRequest {
  string tag_type_id = 1;
  int32 page_size = 2;
  string page_token = 3;
  optional int32 status = 4;
}

// --- 标签值 ---
message TagValuePB {
  string id = 1;
  string tag_group_id = 2;
  string name = 3;
  string description = 4;
  int32 status = 5;
  int32 sort_order = 6;
  int32 version = 7;
  string created_at = 8;
  string updated_at = 9;
}

message CreateTagValueRequest {
  string tag_group_id = 1;
  string name = 2;
  string description = 3;
  int32 sort_order = 4;
}

// --- 打标 ---
message ApplyTagRequest {
  string entity_type = 1;   // 实体类型
  string entity_id = 2;     // 实体 ID
  string tag_value_id = 3;  // 标签值 ID
  string remark = 4;        // 备注
}

message BatchApplyTagsRequest {
  string entity_type = 1;
  string entity_id = 2;
  repeated string tag_value_ids = 3;
  string remark = 4;
}

message BatchApplyTagsResponse {
  repeated TaggingResponse results = 1;
  int32 success_count = 2;
  int32 failure_count = 3;
}

message BatchApplyTagToEntitiesRequest {
  string entity_type = 1;
  repeated string entity_ids = 2;   // 多个实体
  string tag_value_id = 3;          // 一个标签
  string remark = 4;
}

message RevokeTagRequest {
  string tagging_id = 1;
  string remark = 2;
}

message GetEntityTagsRequest {
  string entity_type = 1;
  string entity_id = 2;
}

message GetEntityTagsResponse {
  string entity_type = 1;
  string entity_id = 2;
  repeated EntityTagPB tags = 3;
}

message EntityTagPB {
  string tagging_id = 1;
  TagValuePB tag_value = 2;     // 展开标签值详情
  TagGroupPB tag_group = 3;     // 展开标签组详情
  TagTypePB tag_type = 4;       // 展开标签类型详情
  int32 source = 5;
  string operator_id = 6;
  double confidence = 7;
  string created_at = 8;
}

// --- 自动打标 ---
message AutoTagRequest {
  string entity_type = 1;
  string entity_id = 2;
  string content = 3;            // 待匹配的文本内容
  map<string, string> context = 4; // 额外上下文 (如 channel, subject)
}

message AutoTagResponse {
  repeated MatchedRulePB matched = 1;
  bool applied = 2;              // 是否已自动应用
}

message MatchedRulePB {
  string rule_id = 1;
  string rule_name = 2;
  string tag_value_id = 3;
  double confidence = 4;
  string engine_type = 5;
}

// --- 规则管理 ---
message RulePB {
  string id = 1;
  string name = 2;
  string description = 3;
  string tag_value_id = 4;
  int32 engine_type = 5;       // 1=BUILTIN, 2=EXTERNAL
  string match_config = 6;     // JSON 字符串
  int32 priority = 7;
  string entity_type = 8;
  int32 status = 9;
  int32 version = 10;
  string created_at = 11;
  string updated_at = 12;
}

message CreateRuleRequest {
  string name = 1;
  string description = 2;
  string tag_value_id = 3;
  int32 engine_type = 4;
  string match_config = 5;
  int32 priority = 6;
  string entity_type = 7;
}

// --- 统计 ---
message GetTaggingStatsRequest {
  optional string entity_type = 1;
  optional string tag_value_id = 2;
  optional string start_time = 3;
  optional string end_time = 4;
}

message GetTaggingStatsResponse {
  int64 total_count = 1;
  int64 active_count = 2;
  int64 revoked_count = 3;
  int64 auto_count = 4;
  int64 manual_count = 5;
  repeated TagValueStatPB by_tag_value = 6;
}

message TagValueStatPB {
  string tag_value_id = 1;
  string tag_value_name = 2;
  int64 count = 3;
}

// --- 通用响应 ---
message TagTypeResponse { TagTypePB data = 1; }
message TagGroupResponse { TagGroupPB data = 1; }
message TagValueResponse { TagValuePB data = 1; }
message TaggingResponse { EntityTagPB data = 1; }
message RuleResponse { RulePB data = 1; }
message ListTagValuesResponse { repeated TagValuePB items = 1; string next_page_token = 2; int32 total = 3; }
message ListRulesResponse { repeated RulePB items = 1; string next_page_token = 2; int32 total = 3; }
message GetValueChangeLogRequest { string tag_value_id = 1; int32 page_size = 2; string page_token = 3; }
message GetValueChangeLogResponse { repeated ChangeLogPB items = 1; string next_page_token = 2; }
message ChangeLogPB { string id = 1; int32 old_version = 2; int32 new_version = 3; string change_type = 4; string old_value = 5; string new_value = 6; string operator_id = 7; string created_at = 8; }
```

### 5.3 HTTP 路由映射（Connect 自动生成）

| RPC | HTTP Method | Path |
|---|---|---|
| CreateTagType | POST | /tagsense.v1.TagSenseService/CreateTagType |
| ListTagTypes | POST | /tagsense.v1.TagSenseService/ListTagTypes |
| GetTagType | POST | /tagsense.v1.TagSenseService/GetTagType |
| UpdateTagType | POST | /tagsense.v1.TagSenseService/UpdateTagType |
| ApplyTag | POST | /tagsense.v1.TagSenseService/ApplyTag |
| AutoTag | POST | /tagsense.v1.TagSenseService/AutoTag |
| ... | ... | ... |

Connect 协议同时支持 gRPC 二进制和 HTTP+JSON，客户端可任选。

---

## 六、核心流程设计

### 6.1 手动打标流程

```
Client ──► L5-Gateway (Connect Handler)
              │
              ▼
           L3-Authz (校验身份 + 权限, 调用 auth-center)
              │
              ▼
           L4-Service::ApplyTag
              │
              ├── 1. 校验 TagValue 存在且 ACTIVE
              ├── 2. 校验 TagType 适用范围 (entity_type 匹配)
              ├── 3. 调用 L2-Domain 创建 Tagging 实体
              │       └── 校验唯一约束 (幂等)
              ├── 4. 通过 L1-Storage 持久化 (事务内)
              │       └── 同时写入 Outbox 事件
              └── 5. 刷新 Redis 缓存
```

### 6.2 自动打标流程（双通路）

```
Client ──► AutoTag RPC
              │
              ▼
           L4-Service::AutoTag
              │
              ├── 1. 查询所有 ACTIVE 且匹配 entity_type 的规则
              │
              ├── 2. 并发执行所有规则匹配 (errgroup, 限制并发数)
              │       │
              │       ├── BUILTIN 引擎:
              │       │     KeywordMatcher.Match(content, rule.MatchConfig)
              │       │     └── 返回 confidence
              │       │
              │       └── EXTERNAL 引擎:
              │             ExternalClient.Call(ctx, rule.MatchConfig.Endpoint, content)
              │             └── 返回 matched + confidence
              │
              ├── 3. 收集所有匹配结果, 按优先级取最高
              │
              └── 4. 调用 ApplyTag 完成打标 (复用手动打标逻辑)
```

**规则引擎接口定义（L4-Service/interfaces.go）**：

```go
package service

import "context"

// RuleMatcher 规则匹配器接口 (核心层定义)
type RuleMatcher interface {
    // Match 执行规则匹配, 返回是否匹配 + 置信度
    Match(ctx context.Context, content string, config MatchConfig) (matched bool, confidence float64, err error)
}

// BUILTIN 实现在 plugins/ruleengine/builtin/
// EXTERNAL 实现在 plugins/ruleengine/external/

// RuleEngineService 规则引擎服务接口
type RuleEngineService interface {
    // Evaluate 对内容执行所有匹配规则, 返回匹配结果
    Evaluate(ctx context.Context, req EvaluateRequest) (*EvaluateResponse, error)
}

type EvaluateRequest struct {
    EntityType string
    EntityID   string
    Content    string
    Context    map[string]string
}

type EvaluateResponse struct {
    MatchedRules []MatchedRule
}

type MatchedRule struct {
    RuleID      string
    TagValueID  string
    Confidence  float64
    EngineType  string
}
```

### 6.3 标签值更新流程

```
UpdateTagValue RPC
    │
    ├── 仅名称/描述/排序变更:
    │     └── 原地 UPDATE, 不递增 version
    │         └── 写入 changelog (change_type = NAME_CHANGE / DESC_CHANGE)
    │
    └── 语义变更 (需手动标记 is_semantic_change = true):
          └── UPDATE + version 递增
              └── 写入 changelog (change_type = SEMANTIC_CHANGE)
              └── 发布 TagValueUpdatedV1 事件 (通过 Outbox)
```

---

## 七、领域事件设计

### 7.1 事件类型

| 事件类型 | 触发时机 | Payload 关键字段 |
|---|---|---|
| `TagTypeCreatedV1` | 创建标签类型 | id, name, apply_subjects, apply_channels |
| `TagTypeUpdatedV1` | 更新标签类型 | id, changed_fields |
| `TagGroupCreatedV1` | 创建标签组 | id, tag_type_id, name |
| `TagValueCreatedV1` | 创建标签值 | id, tag_group_id, name |
| `TagValueUpdatedV1` | 语义变更标签值 | id, old_version, new_version |
| `TaggingAppliedV1` | 完成打标 | id, entity_type, entity_id, tag_value_id, source |
| `TaggingRevokedV1` | 撤销打标 | id, entity_type, entity_id, tag_value_id |
| `RuleMatchedV1` | 规则匹配成功 | rule_id, entity_type, entity_id, confidence |

### 7.2 事件流转

```
L2-Domain (事务内收集事件)
    ↓
L1-Storage (写入 outbox_events 表, 同事务)
    ↓
Outbox Poller (后台 goroutine, 1s 轮询)
    ↓
Redis Stream (发布事件)
    ↓
外部消费者 (各业务系统)
```

---

## 八、错误码设计

| 错误码 | 层 | 说明 |
|---|---|---|
| L1_001 | Storage | 数据库连接失败 |
| L1_002 | Storage | 唯一约束冲突 |
| L1_003 | Storage | 外键约束违反 |
| L1_010 | Storage | Outbox 轮询失败 |
| L2_201 | Domain | 状态转换非法 (如 DISABLED 状态下不允许操作) |
| L2_202 | Domain | 标签名称重复 |
| L2_203 | Domain | 业务规则校验失败 (适用范围不匹配) |
| L2_204 | Domain | 乐观锁版本冲突 |
| L3_401 | Authz | 权限拒绝 |
| L3_402 | Authz | 身份验证失败 (auth-center 返回未授权) |
| L4_601 | Service | 输入校验失败 |
| L4_602 | Service | 规则引擎匹配失败 |
| L4_603 | Service | 外部规则服务调用超时 |
| L5_801 | Gateway | 请求参数解析失败 |
| L5_802 | Gateway | 内部服务错误 |

---

## 九、与 auth-center 对接设计

### 9.1 对接方式

本项目 L3-Authz 层通过 HTTP/gRPC 调用 auth-center 进行权限校验：

```go
// L3-Authz: auth-center 客户端接口
type AuthCenterClient interface {
    // CheckPermission 检查用户是否有指定权限
    CheckPermission(ctx context.Context, req CheckPermissionRequest) (bool, error)
}

type CheckPermissionRequest struct {
    UserID   string
    Resource string  // 如 "tag_type", "tag_value", "rule"
    Action   string  // 如 "create", "update", "delete", "apply"
}
```

### 9.2 Connect 拦截器

L5-Gateway 注册 Auth 拦截器，每个 RPC 请求经过：

1. 解析 JWT Token → 提取 user_id
2. 调用 auth-center CheckPermission → 判断是否有权限
3. 通过 → 将 user_context 注入 ctx，传递到 L4
4. 拒绝 → 返回 Permission Denied 错误

### 9.3 降级策略

auth-center 不可用时的处理：

- 权限结果缓存到 Redis（Key: `auth:perm:{user_id}:{resource}:{action}`，TTL 5min）
- auth-center 调用失败 → 先查缓存，命中则放行
- 缓存也未命中 → 返回 503 Service Unavailable（Fail Closed，安全优先）
- 连续失败超过阈值（如 30s 内 10 次）→ 熔断，直接走缓存/拒绝，避免雪崩

### 9.4 资源-动作映射

| RPC | Resource | Action |
|---|---|---|
| CreateTagType | tag_type | create |
| UpdateTagType | tag_type | update |
| DisableTagType | tag_type | disable |
| ApplyTag | tagging | apply |
| AutoTag | tagging | auto_apply |
| CreateRule | rule | create |
| DisableRule | rule | disable |
| ... | ... | ... |

---

## 十、缓存与性能

### 10.1 缓存流程

```
查询请求 → 查 Redis 缓存
              ├── 命中 → 返回
              └── 未命中 → 查 PostgreSQL
                            ├── 写入 Redis (设置 TTL)
                            └── 返回
```

### 10.2 缓存一致性

- **写操作**：更新 DB 后，同步删除/更新对应 Redis Key
- **Outbox 事件**：可作为缓存失效的兜底机制
- **Trade-off**：DB 更新与 Redis 删除之间存在微小窗口（<1ms），并发读可能拿到旧值。标签数据变更频率极低，此竞态可接受

### 10.3 性能目标

| 操作 | 目标响应时间 |
|---|---|
| 单条打标 | ≤ 100ms |
| 批量打标 (100条) | ≤ 1s |
| 标签查询 | ≤ 50ms (缓存命中) |
| 自动打标 (单条) | ≤ 200ms (内置) / ≤ 500ms (外部) |
| 实体标签列表 | ≤ 100ms |

---

## 十一、待定与扩展

### 11.1 后续迭代方向

- **分库分表**：当 tagging 表数据量达到单表瓶颈时，按 entity_type 分区
- **OpenFGA 细粒度权限**：当前通过 auth-center 统一管理，后续可按主体/渠道细化
- **标签导入导出**：批量导入标签体系配置
- **打标审核流**：自动打标结果的人工审核机制
- **前端 SDK**：独立设计文档，提供标签选择器等组件

### 11.2 与 PDF 原始设计的差异说明

| 差异点 | PDF 设计 | 本设计 | 原因 |
|---|---|---|---|
| 数据库 | MySQL + MongoDB | PostgreSQL | Go 技术栈统一 |
| 用户/角色管理 | 内置 sys_user/sys_role | 对接 auth-center | 避免重复造轮子 |
| 版本控制 | 新增行 + is_latest | 原地更新 + changelog | 简化模型, tag_relation 关联 ID |
| 分库分表 | 按主体+月份分库分表 | 暂不实施 | 初期数据量不需要 |
| 规则引擎 | 独立微服务 + RocketMQ | 内置 + 外部接口双通路 | 适配 Go 框架, 降低复杂度 |
| 缓存 | Redis + Caffeine 多级 | Redis 单级 | 标签数据量有限, 单级足够 |
