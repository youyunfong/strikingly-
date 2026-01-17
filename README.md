# 投票系统数据库表设计文档（Rails 8.1.2 落地版）
## 文档说明
本文档基于投票系统原始业务需求，结合 Rails 8.1.2 + MySQL 环境的实际兼容规则调整，解决了外键类型不匹配、ENUM/UNSIGNED 语法兼容等问题，所有数据库层面无法兼容的约束均移至模型层验证。

## 1. 投票者模块（voters 表）| 登录/注册页
| 字段名称         | 数据类型（MySQL）| 导轨类型  | 约束条件                                                                 | 关联网页场景                     |
|------------------|---------------------|-----------|--------------------------------------------------------------------------|----------------------------------|
| id               | BIGINT              | bigint    | 主键、自增、NOT NULL（Rails 默认）| 所有场景（唯一标识投票者）|
| username         | VARCHAR(20)         | string    | NOT NULL、唯一、6-20字符、仅字母+数字（模型层验证）| 登录页/注册页（用户名验证）|
| password_digest  | VARCHAR(255)        | string    | NOT NULL                                                                 | 登录页/注册页（密码存储）|
| cell_phone       | VARCHAR(15)         | string    | NOT NULL、唯一、8-15个字符（模型层验证）| 注册页（验证码发送）|
| gender           | VARCHAR(6)          | string    | NOT NULL、仅限male/female（模型层验证，替代ENUM）| 注册页（性别选择）|
| year_of_birth    | INT                 | integer   | NOT NULL、4位整数、1900-2026（模型层验证，替代INT(4) unsigned）| 注册页（出生年份输入）|
| created_at       | DATETIME(6)         | datetime  | NOT NULL（Rails 默认）| 系统日志（无前端展示）|
| updated_at       | DATETIME(6)         | datetime  | NOT NULL（Rails 默认）| 系统日志（无前端展示）|

## 2. 验证码模块（verification_codes 表）| 注册页
| 字段名称     | 数据类型（MySQL）| 导轨类型  | 约束条件                                   | 关联网页场景               |
|--------------|---------------------|-----------|--------------------------------------------|----------------------------|
| id           | BIGINT              | bigint    | 主键、自增、NOT NULL（Rails 默认）| 系统内部标识               |
| cell_phone   | VARCHAR(15)         | string    | NOT NULL、8-15 个字符（模型层验证）| 注册页（手机号关联）|
| code         | VARCHAR(6)          | string    | NOT NULL、6位字符（模型层验证）| 注册页（验证码输入）|
| expired_at   | DATETIME(6)         | datetime  | NOT NULL                                   | 注册页（验证码有效校验）|
| created_at   | DATETIME(6)         | datetime  | NOT NULL（Rails 默认）| 系统日志                   |

## 3. 竞选模块（candidates 表）| 投票列表页/投票页
| 字段名称      | 数据类型（MySQL）| 导轨类型  | 约束条件                                                                 | 关联网页场景               |
|---------------|---------------------|-----------|--------------------------------------------------------------------------|----------------------------|
| id            | BIGINT              | bigint    | 主键、自增、NOT NULL（Rails 默认）| 列表页（按ID降序分页）|
| name          | VARCHAR(200)        | string    | NOT NULL、最多200个字符                                                 | 列表页/投票页（姓名展示）|
| age           | INT                 | integer   | NOT NULL、正整数（模型层验证，替代TINYINT unsigned）| 列表页/投票页（年龄显示）|
| video_url     | VARCHAR(255)        | string    | NOT NULL                                                                 | 列表页/投票页（视频链接）|
| introduction  | TEXT                | text      | NOT NULL                                                                 | 列表页/投票页（介绍文本）|
| vote_count    | INT                 | integer   | NOT NULL、默认0、非负整数（模型层验证，替代INT unsigned）| 投票页（票数统计）|
| created_at    | DATETIME(6)         | datetime  | NOT NULL（Rails 默认）| 系统日志                   |
| updated_at    | DATETIME(6)         | datetime  | NOT NULL（Rails 默认）| 系统日志                   |

## 4. 竞选照片模块（candidate_photos 表）| 竞选列表页
| 字段名称       | 数据类型（MySQL）| 导轨类型  | 约束条件                                                               | 关联网页场景               |
|----------------|---------------------|-----------|------------------------------------------------------------------------|----------------------------|
| id             | BIGINT              | bigint    | 主键、自增、NOT NULL（Rails 默认）| 系统内部标识               |
| candidate_id   | BIGINT              | bigint    | NOT NULL、外键（关联candidates.id，类型匹配BIGINT）| 列表页（照片关联候选人）|
| photo_url      | VARCHAR(255)        | string    | NOT NULL                                                               | 列表页（4行5列图片展示）|
| created_at     | DATETIME(6)         | datetime  | NOT NULL（Rails 默认）| 系统日志                   |

## 5. 投票记录模块（votes 表）| 投票页
| 字段名称       | 数据类型（MySQL）| 导轨类型  | 约束条件                                                                 | 关联网页场景               |
|----------------|---------------------|-----------|--------------------------------------------------------------------------|----------------------------|
| id             | BIGINT              | bigint    | 主键、自增、NOT NULL（Rails 默认）| 系统内部标识               |
| voter_id       | BIGINT              | bigint    | NOT NULL、外键（关联voters.id）、唯一索引（1人1票）| 投票页（1人1票约束）|
| candidate_id   | BIGINT              | bigint    | NOT NULL、外键（关联candidates.id）| 投票页（关联被投候选人）|
| created_at     | DATETIME(6)         | datetime  | NOT NULL（Rails 默认）| 系统日志（投票时间）|

## 核心更新说明
### 1. 类型统一
- 所有表主键 `id` 改为 Rails 8.1.2 默认的 `BIGINT`（导轨类型 `bigint`）；
- 外键字段（`candidate_id`/`voter_id`）同步改为 `BIGINT`，解决外键类型不匹配问题。

### 2. 兼容型约束调整
- 移除 `unsigned` 约束：移至模型层验证数值非负/正；
- 移除 `ENUM` 类型：改用 `VARCHAR` + 模型层枚举值验证；
- 移除 `TINYINT`/`INT(4)` 等特定长度约束：模型层验证数值范围。

### 3. 保留核心约束
- 数据库层保留 `NOT NULL`、唯一索引、外键约束，保证数据基础合法性；
- 模型层补充格式、范围、枚举等业务约束，兼顾兼容性和业务规则。
