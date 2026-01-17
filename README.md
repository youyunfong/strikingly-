字段名称	数据类型（MySQL）	导轨类型	约束条件	关联网页场景
id	BIGINT	bigint	主键、自增、NOT NULL（Rails 默认）	所有场景（唯一标识投票者）
username	VARCHAR(20)	string	NOT NULL、唯一、6-20 字符、仅字母 + 数字（模型层验证）	登录页 / 注册页（用户名验证）
password_digest	VARCHAR(255)	string	NOT NULL	登录页 / 注册页（密码存储）
cell_phone	VARCHAR(15)	string	NOT NULL、唯一、8-15 个字符（模型层验证）	注册页（验证码发送）
gender	VARCHAR(6)	string	NOT NULL、仅限 male/female（模型层验证，替代 ENUM）	注册页（性别选择）
year_of_birth	INT	integer	NOT NULL、4 位整数、1900-2026（模型层验证，替代 INT (4) unsigned）	注册页（出生年份输入）
created_at	DATETIME(6)	datetime	NOT NULL（Rails 默认）	系统日志（无前端展示）
updated_at	DATETIME(6)	datetime	NOT NULL（Rails 默认）	系统日志（无前端展示）
2. 验证码模块（verification_codes 表）| 注册页
字段名称	数据类型（MySQL）	导轨类型	约束条件	关联网页场景
id	BIGINT	bigint	主键、自增、NOT NULL（Rails 默认）	系统内部标识
cell_phone	VARCHAR(15)	string	NOT NULL、8-15 个字符（模型层验证）	注册页（手机号关联）
code	VARCHAR(6)	string	NOT NULL、6 位字符（模型层验证）	注册页（验证码输入）
expired_at	DATETIME(6)	datetime	NOT NULL	注册页（验证码有效校验）
created_at	DATETIME(6)	datetime	NOT NULL（Rails 默认）	系统日志
3. 竞选模块（candidates 表）| 投票列表页 / 投票页
字段名称	数据类型（MySQL）	导轨类型	约束条件	关联网页场景
id	BIGINT	bigint	主键、自增、NOT NULL（Rails 默认）	列表页（按 ID 降序分页）
name	VARCHAR(200)	string	NOT NULL、最多 200 个字符	列表页 / 投票页（姓名展示）
age	INT	integer	NOT NULL、正整数（模型层验证，替代 TINYINT unsigned）	列表页 / 投票页（年龄显示）
video_url	VARCHAR(255)	string	NOT NULL	列表页 / 投票页（视频链接）
introduction	TEXT	text	NOT NULL	列表页 / 投票页（介绍文本）
vote_count	INT	integer	NOT NULL、默认 0、非负整数（模型层验证，替代 INT unsigned）	投票页（票数统计）
created_at	DATETIME(6)	datetime	NOT NULL（Rails 默认）	系统日志
updated_at	DATETIME(6)	datetime	NOT NULL（Rails 默认）	系统日志
4. 竞选照片模块（candidate_photos 表）| 竞选列表页
字段名称	数据类型（MySQL）	导轨类型	约束条件	关联网页场景
id	BIGINT	bigint	主键、自增、NOT NULL（Rails 默认）	系统内部标识
candidate_id	BIGINT	bigint	NOT NULL、外键（关联 candidates.id，类型匹配 BIGINT，替代 INT unsigned）	列表页（照片关联候选人）
photo_url	VARCHAR(255)	string	NOT NULL	列表页（4 行 5 列图片展示）
created_at	DATETIME(6)	datetime	NOT NULL（Rails 默认）	系统日志
5. 投票记录模块（votes 表）| 投票页
字段名称	数据类型（MySQL）	导轨类型	约束条件	关联网页场景
id	BIGINT	bigint	主键、自增、NOT NULL（Rails 默认）	系统内部标识
voter_id	BIGINT	bigint	NOT NULL、外键（关联 voters.id，类型匹配 BIGINT）、唯一索引（1 人 1 票）	投票页（1 人 1 票约束）
candidate_id	BIGINT	bigint	NOT NULL、外键（关联 candidates.id，类型匹配 BIGINT，替代 INT unsigned）	投票页（关联被投候选人）
created_at	DATETIME(6)	datetime	NOT NULL（Rails 默认）
