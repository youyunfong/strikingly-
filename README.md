# 香港选美投票网站 - 数据库字段设计
## 说明
本文件汇总了网站所有核心模块的数据库字段名称、类型、约束条件及关联网页场景，适配 MySQL + Ruby on Rails 技术栈。

## 1. 投票者模块（voters 表）| 登录/注册页
| 字段名称         | 数据类型（MySQL）| Rails 类型 | 约束条件                                  | 关联网页场景                  |
|------------------|---------------------|------------|-------------------------------------------|-------------------------------|
| `id`             | INT UNSIGNED        | integer    | 主键、自增、NOT NULL                      | 所有场景（唯一标识投票者）|
| `username`       | VARCHAR(20)         | string     | NOT NULL、唯一、6-20字符、仅字母+数字     | 登录页/注册页（用户名校验）|
| `password_digest`| VARCHAR(255)        | string     | NOT NULL                                  | 登录页/注册页（密码存储）|
| `cell_phone`     | VARCHAR(15)         | string     | NOT NULL、唯一、8-15字符                  | 注册页（验证码发送）|
| `gender`         | ENUM('male','female') | enum       | NOT NULL、仅男/女                         | 注册页（性别选择）|
| `year_of_birth`  | INT(4) UNSIGNED     | integer    | NOT NULL、4位整数、1900-2026              | 注册页（出生年份输入）|
| `created_at`     | DATETIME            | datetime   | NOT NULL                                  | 系统日志（无前端展示）|
| `updated_at`     | DATETIME            | datetime   | NOT NULL                                  | 系统日志（无前端展示）|

## 2. 验证码模块（verification_codes 表）| 注册页
| 字段名称         | 数据类型（MySQL）| Rails 类型 | 约束条件                                  | 关联网页场景                  |
|------------------|---------------------|------------|-------------------------------------------|-------------------------------|
| `id`             | INT UNSIGNED        | integer    | 主键、自增、NOT NULL                      | 系统内部标识                  |
| `cell_phone`     | VARCHAR(15)         | string     | NOT NULL、8-15字符                        | 注册页（手机号关联）|
| `code`           | VARCHAR(6)          | string     | NOT NULL、6位字符                         | 注册页（验证码输入）|
| `expired_at`     | DATETIME            | datetime   | NOT NULL                                  | 注册页（验证码有效期校验）|
| `created_at`     | DATETIME            | datetime   | NOT NULL                                  | 系统日志                      |

## 3. 候选人模块（candidates 表）| 候选人列表页/投票页
| 字段名称         | 数据类型（MySQL）| Rails 类型 | 约束条件                                  | 关联网页场景                  |
|------------------|---------------------|------------|-------------------------------------------|-------------------------------|
| `id`             | INT UNSIGNED        | integer    | 主键、自增、NOT NULL                      | 列表页（按ID降序分页）|
| `name`           | VARCHAR(200)        | string     | NOT NULL、最长200字符                     | 列表页/投票页（姓名展示）|
| `age`            | TINYINT UNSIGNED    | integer    | NOT NULL、正整数                          | 列表页/投票页（年龄展示）|
| `video_url`      | VARCHAR(255)        | string     | NOT NULL                                  | 列表页/投票页（视频链接）|
| `introduction`   | TEXT                | text       | NOT NULL                                  | 列表页/投票页（介绍文本）|
| `vote_count`     | INT UNSIGNED        | integer    | NOT NULL、默认0、非负整数                 | 投票页（票数统计）|
| `created_at`     | DATETIME            | datetime   | NOT NULL                                  | 系统日志                      |
| `updated_at`     | DATETIME            | datetime   | NOT NULL                                  | 系统日志                      |

## 4. 候选人照片模块（candidate_photos 表）| 候选人列表页
| 字段名称         | 数据类型（MySQL）| Rails 类型 | 约束条件                                  | 关联网页场景                  |
|------------------|---------------------|------------|-------------------------------------------|-------------------------------|
| `id`             | INT UNSIGNED        | integer    | 主键、自增、NOT NULL                      | 系统内部标识                  |
| `candidate_id`   | INT UNSIGNED        | integer    | NOT NULL、外键（关联candidates.id）| 列表页（照片归属候选人）|
| `photo_url`      | VARCHAR(255)        | string     | NOT NULL                                  | 列表页（4行5列图片展示）|
| `created_at`     | DATETIME            | datetime   | NOT NULL                                  | 系统日志                      |

## 5. 投票记录模块（votes 表）| 投票页
| 字段名称         | 数据类型（MySQL）| Rails 类型 | 约束条件                                  | 关联网页场景                  |
|------------------|---------------------|------------|-------------------------------------------|-------------------------------|
| `id`             | INT UNSIGNED        | integer    | 主键、自增、NOT NULL                      | 系统内部标识                  |
| `voter_id`       | INT UNSIGNED        | integer    | NOT NULL、外键、唯一                      | 投票页（1人1票约束）|
| `candidate_id`   | INT UNSIGNED        | integer    | NOT NULL、外键                            | 投票页（关联被投候选人）|
| `created_at`     | DATETIME            | datetime   | NOT NULL                                  | 系统日志（投票时间）|
