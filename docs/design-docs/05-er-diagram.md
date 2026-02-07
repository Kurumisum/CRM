# 电信CRM客户中心 - 数据库ER图

## 一、概述

本文档描述客户中心各微服务的数据库设计，遵循DDD原则，每个限界上下文拥有独立的数据库，避免跨库JOIN。

## 二、数据库设计原则

### 2.1 DDD数据库设计原则

1. **每个限界上下文独立数据库**：避免跨上下文的数据库依赖
2. **聚合作为事务边界**：一个聚合的数据在同一个事务中修改
3. **通过ID引用其他聚合**：不使用外键关联其他聚合
4. **事件驱动同步数据**：通过领域事件保持数据最终一致性

### 2.2 性能优化原则

1. **读写分离**：主库写入，从库读取
2. **分库分表**：按客户ID或用户ID进行水平拆分
3. **冗余设计**：适当冗余常用字段，避免跨库查询
4. **索引优化**：在高频查询字段上建立索引

## 三、客户服务数据库（customer_db）

### 3.1 客户表（customer）

**表说明**：存储客户基本信息

| 字段名 | 类型 | 长度 | 约束 | 说明 |
|-------|------|------|------|------|
| customer_id | BIGINT | - | PK | 客户ID（主键） |
| customer_type | VARCHAR | 20 | NOT NULL | 客户类型：INDIVIDUAL/FAMILY/ENTERPRISE |
| status | VARCHAR | 20 | NOT NULL | 客户状态：ACTIVE/ARREARS/SUSPENDED/CLOSED |
| level | INT | - | NOT NULL | 客户等级（1-5星） |
| points | INT | - | DEFAULT 0 | 客户积分 |
| version | BIGINT | - | NOT NULL | 版本号（乐观锁） |
| created_time | DATETIME | - | NOT NULL | 创建时间 |
| updated_time | DATETIME | - | NOT NULL | 更新时间 |
| created_by | VARCHAR | 50 | NOT NULL | 创建人 |
| updated_by | VARCHAR | 50 | NOT NULL | 更新人 |

**索引**：
- PRIMARY KEY (`customer_id`)
- INDEX `idx_status` (`status`)
- INDEX `idx_created_time` (`created_time`)

### 3.2 个人客户档案表（individual_customer_profile）

**表说明**：存储个人客户的详细信息

| 字段名 | 类型 | 长度 | 约束 | 说明 |
|-------|------|------|------|------|
| customer_id | BIGINT | - | PK | 客户ID（主键，关联customer表） |
| name | VARCHAR | 100 | NOT NULL | 姓名 |
| id_type | VARCHAR | 20 | NOT NULL | 证件类型 |
| id_number | VARCHAR | 50 | NOT NULL | 证件号码 |
| gender | VARCHAR | 10 | - | 性别 |
| birth_date | DATE | - | - | 出生日期 |
| contact_phone | VARCHAR | 20 | NOT NULL | 联系电话 |
| email | VARCHAR | 100 | - | 电子邮箱 |
| province | VARCHAR | 50 | - | 省份 |
| city | VARCHAR | 50 | - | 城市 |
| district | VARCHAR | 50 | - | 区县 |
| street | VARCHAR | 100 | - | 街道 |
| detail_address | VARCHAR | 200 | - | 详细地址 |
| postal_code | VARCHAR | 10 | - | 邮政编码 |

**索引**：
- PRIMARY KEY (`customer_id`)
- UNIQUE INDEX `uk_id_number` (`id_number`)
- INDEX `idx_name` (`name`)

### 3.3 家庭客户档案表（family_customer_profile）

**表说明**：存储家庭客户的详细信息

| 字段名 | 类型 | 长度 | 约束 | 说明 |
|-------|------|------|------|------|
| customer_id | BIGINT | - | PK | 客户ID（主键） |
| family_name | VARCHAR | 100 | NOT NULL | 家庭名称 |
| head_member_name | VARCHAR | 100 | NOT NULL | 户主姓名 |
| head_id_number | VARCHAR | 50 | NOT NULL | 户主证件号 |
| contact_phone | VARCHAR | 20 | NOT NULL | 联系电话 |
| province | VARCHAR | 50 | - | 省份 |
| city | VARCHAR | 50 | - | 城市 |
| district | VARCHAR | 50 | - | 区县 |
| street | VARCHAR | 100 | - | 街道 |
| detail_address | VARCHAR | 200 | - | 详细地址 |

**索引**：
- PRIMARY KEY (`customer_id`)
- UNIQUE INDEX `uk_head_id_number` (`head_id_number`)

### 3.4 家庭成员表（family_member）

**表说明**：存储家庭客户的成员信息

| 字段名 | 类型 | 长度 | 约束 | 说明 |
|-------|------|------|------|------|
| member_id | BIGINT | - | PK | 成员ID（主键） |
| customer_id | BIGINT | - | NOT NULL | 客户ID（家庭客户） |
| member_name | VARCHAR | 100 | NOT NULL | 成员姓名 |
| member_id_number | VARCHAR | 50 | NOT NULL | 成员证件号 |
| relationship | VARCHAR | 20 | NOT NULL | 与户主关系 |
| contact_phone | VARCHAR | 20 | - | 联系电话 |

**索引**：
- PRIMARY KEY (`member_id`)
- INDEX `idx_customer_id` (`customer_id`)
- INDEX `idx_id_number` (`member_id_number`)

### 3.5 政企客户档案表（enterprise_customer_profile）

**表说明**：存储政企客户的详细信息

| 字段名 | 类型 | 长度 | 约束 | 说明 |
|-------|------|------|------|------|
| customer_id | BIGINT | - | PK | 客户ID（主键） |
| enterprise_name | VARCHAR | 200 | NOT NULL | 企业名称 |
| organization_code | VARCHAR | 50 | NOT NULL | 组织机构代码 |
| tax_number | VARCHAR | 50 | NOT NULL | 税号 |
| legal_person | VARCHAR | 100 | NOT NULL | 法人代表 |
| contact_person | VARCHAR | 100 | NOT NULL | 联系人 |
| contact_phone | VARCHAR | 20 | NOT NULL | 联系电话 |
| province | VARCHAR | 50 | - | 省份 |
| city | VARCHAR | 50 | - | 城市 |
| district | VARCHAR | 50 | - | 区县 |
| street | VARCHAR | 100 | - | 街道 |
| business_address | VARCHAR | 200 | - | 营业地址 |

**索引**：
- PRIMARY KEY (`customer_id`)
- UNIQUE INDEX `uk_organization_code` (`organization_code`)
- UNIQUE INDEX `uk_tax_number` (`tax_number`)

### 3.6 客户变更历史表（customer_history）

**表说明**：记录客户的所有变更历史

| 字段名 | 类型 | 长度 | 约束 | 说明 |
|-------|------|------|------|------|
| history_id | BIGINT | - | PK | 历史记录ID（主键） |
| customer_id | BIGINT | - | NOT NULL | 客户ID |
| change_type | VARCHAR | 50 | NOT NULL | 变更类型 |
| old_value | JSON | - | - | 变更前的值 |
| new_value | JSON | - | - | 变更后的值 |
| change_time | DATETIME | - | NOT NULL | 变更时间 |
| change_by | VARCHAR | 50 | NOT NULL | 变更人 |

**索引**：
- PRIMARY KEY (`history_id`)
- INDEX `idx_customer_id` (`customer_id`)
- INDEX `idx_change_time` (`change_time`)

## 四、用户服务数据库（user_db）

### 4.1 用户表（user）

**表说明**：存储用户基本信息

| 字段名 | 类型 | 长度 | 约束 | 说明 |
|-------|------|------|------|------|
| user_id | BIGINT | - | PK | 用户ID（主键） |
| phone_number | VARCHAR | 11 | NOT NULL | 手机号码（11位） |
| customer_id | BIGINT | - | NOT NULL | 所属客户ID（冗余） |
| user_type | VARCHAR | 20 | NOT NULL | 用户类型：INDIVIDUAL/FAMILY_GROUP/ENTERPRISE_GROUP |
| status | VARCHAR | 30 | NOT NULL | 用户状态 |
| open_time | DATETIME | - | NOT NULL | 开户时间 |
| active_time | DATETIME | - | - | 激活时间 |
| version | BIGINT | - | NOT NULL | 版本号（乐观锁） |
| created_time | DATETIME | - | NOT NULL | 创建时间 |
| updated_time | DATETIME | - | NOT NULL | 更新时间 |

**索引**：
- PRIMARY KEY (`user_id`)
- UNIQUE INDEX `uk_phone_number` (`phone_number`)
- INDEX `idx_customer_id` (`customer_id`)
- INDEX `idx_status` (`status`)
- INDEX `idx_open_time` (`open_time`)

**分表策略**：按user_id取模分256张表
- user_000 ~ user_255

### 4.2 SIM卡表（sim_card）

**表说明**：存储SIM卡信息

| 字段名 | 类型 | 长度 | 约束 | 说明 |
|-------|------|------|------|------|
| sim_card_id | BIGINT | - | PK | SIM卡ID（主键） |
| user_id | BIGINT | - | NOT NULL | 用户ID |
| iccid | VARCHAR | 20 | NOT NULL | ICCID（20位） |
| imsi | VARCHAR | 15 | NOT NULL | IMSI（15位） |
| card_type | VARCHAR | 10 | NOT NULL | 卡类型：2G/3G/4G/5G |
| status | VARCHAR | 20 | NOT NULL | 卡状态：NORMAL/LOST/DAMAGED/INVALID |
| issue_time | DATETIME | - | NOT NULL | 发卡时间 |
| version | BIGINT | - | NOT NULL | 版本号 |

**索引**：
- PRIMARY KEY (`sim_card_id`)
- UNIQUE INDEX `uk_iccid` (`iccid`)
- UNIQUE INDEX `uk_imsi` (`imsi`)
- INDEX `idx_user_id` (`user_id`)

### 4.3 套餐信息表（service_package）

**表说明**：存储用户当前套餐信息（冗余产商品中心的数据）

| 字段名 | 类型 | 长度 | 约束 | 说明 |
|-------|------|------|------|------|
| id | BIGINT | - | PK | 主键ID |
| user_id | BIGINT | - | NOT NULL | 用户ID |
| package_id | VARCHAR | 50 | NOT NULL | 套餐ID（引用产商品中心） |
| package_name | VARCHAR | 100 | NOT NULL | 套餐名称（冗余） |
| monthly_fee | DECIMAL | 10,2 | NOT NULL | 月租费（冗余） |
| included_traffic_mb | INT | - | - | 包含流量（MB） |
| included_voice_min | INT | - | - | 包含语音（分钟） |
| included_sms | INT | - | - | 包含短信（条） |
| effective_type | VARCHAR | 20 | NOT NULL | 生效类型：IMMEDIATE/NEXT_MONTH |
| effective_time | DATETIME | - | NOT NULL | 生效时间 |
| expiry_time | DATETIME | - | - | 失效时间 |

**索引**：
- PRIMARY KEY (`id`)
- INDEX `idx_user_id` (`user_id`)
- INDEX `idx_effective_time` (`effective_time`)

### 4.4 套餐变更历史表（package_change_history）

**表说明**：记录用户套餐变更历史

| 字段名 | 类型 | 长度 | 约束 | 说明 |
|-------|------|------|------|------|
| history_id | BIGINT | - | PK | 历史记录ID（主键） |
| user_id | BIGINT | - | NOT NULL | 用户ID |
| old_package_id | VARCHAR | 50 | - | 原套餐ID |
| new_package_id | VARCHAR | 50 | NOT NULL | 新套餐ID |
| change_type | VARCHAR | 20 | NOT NULL | 变更类型：IMMEDIATE/NEXT_MONTH |
| change_time | DATETIME | - | NOT NULL | 变更时间 |
| effective_time | DATETIME | - | NOT NULL | 生效时间 |
| change_by | VARCHAR | 50 | NOT NULL | 变更人 |

**索引**：
- PRIMARY KEY (`history_id`)
- INDEX `idx_user_id` (`user_id`)
- INDEX `idx_change_time` (`change_time`)

### 4.5 用户状态变更历史表（user_status_history）

**表说明**：记录用户状态变更历史

| 字段名 | 类型 | 长度 | 约束 | 说明 |
|-------|------|------|------|------|
| history_id | BIGINT | - | PK | 历史记录ID（主键） |
| user_id | BIGINT | - | NOT NULL | 用户ID |
| old_status | VARCHAR | 30 | - | 原状态 |
| new_status | VARCHAR | 30 | NOT NULL | 新状态 |
| reason | VARCHAR | 200 | - | 变更原因 |
| change_time | DATETIME | - | NOT NULL | 变更时间 |
| change_by | VARCHAR | 50 | NOT NULL | 变更人 |

**索引**：
- PRIMARY KEY (`history_id`)
- INDEX `idx_user_id` (`user_id`)
- INDEX `idx_change_time` (`change_time`)

## 五、账户服务数据库（account_db）

### 5.1 账户表（account）

**表说明**：存储账户基本信息

| 字段名 | 类型 | 长度 | 约束 | 说明 |
|-------|------|------|------|------|
| account_id | BIGINT | - | PK | 账户ID（主键） |
| customer_id | BIGINT | - | NOT NULL | 所属客户ID（冗余） |
| account_type | VARCHAR | 20 | NOT NULL | 账户类型：PREPAID/POSTPAID/MIXED |
| balance | DECIMAL | 18,2 | NOT NULL | 账户余额（分） |
| frozen_balance | DECIMAL | 18,2 | DEFAULT 0 | 冻结余额（分） |
| credit_limit | DECIMAL | 18,2 | DEFAULT 0 | 信用额度（分） |
| status | VARCHAR | 20 | NOT NULL | 账户状态：ACTIVE/FROZEN/CLOSED |
| open_time | DATETIME | - | NOT NULL | 开户时间 |
| version | BIGINT | - | NOT NULL | 版本号（乐观锁） |
| created_time | DATETIME | - | NOT NULL | 创建时间 |
| updated_time | DATETIME | - | NOT NULL | 更新时间 |

**索引**：
- PRIMARY KEY (`account_id`)
- INDEX `idx_customer_id` (`customer_id`)
- INDEX `idx_status` (`status`)

**分表策略**：按account_id取模分256张表
- account_000 ~ account_255

**备注**：
- 余额单位为分，避免浮点数精度问题
- balance = 可用余额 + frozen_balance

### 5.2 账户交易表（account_transaction）

**表说明**：存储所有账户交易记录

| 字段名 | 类型 | 长度 | 约束 | 说明 |
|-------|------|------|------|------|
| transaction_id | VARCHAR | 50 | PK | 交易流水号（主键） |
| account_id | BIGINT | - | NOT NULL | 账户ID |
| transaction_type | VARCHAR | 20 | NOT NULL | 交易类型 |
| amount | DECIMAL | 18,2 | NOT NULL | 交易金额（分） |
| balance_before | DECIMAL | 18,2 | NOT NULL | 交易前余额（分） |
| balance_after | DECIMAL | 18,2 | NOT NULL | 交易后余额（分） |
| transaction_time | DATETIME | - | NOT NULL | 交易时间 |
| channel | VARCHAR | 50 | - | 交易渠道 |
| description | VARCHAR | 200 | - | 交易描述 |
| related_order_id | VARCHAR | 50 | - | 关联订单号 |

**索引**：
- PRIMARY KEY (`transaction_id`)
- INDEX `idx_account_id_time` (`account_id`, `transaction_time` DESC)
- INDEX `idx_transaction_time` (`transaction_time`)
- INDEX `idx_transaction_type` (`transaction_type`)

**分表策略**：按交易时间按月分表
- account_transaction_202401
- account_transaction_202402
- ...

### 5.3 账户用户关联表（account_user_relationship）

**表说明**：存储账户与用户的关联关系

| 字段名 | 类型 | 长度 | 约束 | 说明 |
|-------|------|------|------|------|
| relationship_id | BIGINT | - | PK | 关联ID（主键） |
| account_id | BIGINT | - | NOT NULL | 账户ID |
| user_id | BIGINT | - | NOT NULL | 用户ID（冗余） |
| relationship_type | VARCHAR | 20 | NOT NULL | 关联类型：PRIMARY/SECONDARY |
| priority | INT | - | DEFAULT 1 | 扣费优先级 |
| effective_time | DATETIME | - | NOT NULL | 生效时间 |
| expiry_time | DATETIME | - | - | 失效时间 |
| created_time | DATETIME | - | NOT NULL | 创建时间 |

**索引**：
- PRIMARY KEY (`relationship_id`)
- INDEX `idx_account_id` (`account_id`)
- INDEX `idx_user_id` (`user_id`)
- UNIQUE INDEX `uk_account_user` (`account_id`, `user_id`)

## 六、产品订购服务数据库（subscription_db）

### 6.1 产品订购表（subscription）

**表说明**：存储产品订购记录

| 字段名 | 类型 | 长度 | 约束 | 说明 |
|-------|------|------|------|------|
| subscription_id | BIGINT | - | PK | 订购ID（主键） |
| user_id | BIGINT | - | NOT NULL | 用户ID |
| product_id | VARCHAR | 50 | NOT NULL | 产品ID（引用产商品中心） |
| product_name | VARCHAR | 100 | NOT NULL | 产品名称（冗余） |
| subscription_type | VARCHAR | 20 | NOT NULL | 订购类型：PACKAGE/VALUE_ADDED/DISCOUNT |
| status | VARCHAR | 20 | NOT NULL | 订购状态 |
| subscribe_time | DATETIME | - | NOT NULL | 订购时间 |
| effective_time | DATETIME | - | NOT NULL | 生效时间 |
| expiry_time | DATETIME | - | - | 失效时间 |
| version | BIGINT | - | NOT NULL | 版本号 |
| created_time | DATETIME | - | NOT NULL | 创建时间 |
| updated_time | DATETIME | - | NOT NULL | 更新时间 |

**索引**：
- PRIMARY KEY (`subscription_id`)
- INDEX `idx_user_id` (`user_id`)
- INDEX `idx_product_id` (`product_id`)
- INDEX `idx_status` (`status`)
- INDEX `idx_effective_time` (`effective_time`)

### 6.2 订购历史表（subscription_history）

**表说明**：记录产品订购的所有变更历史

| 字段名 | 类型 | 长度 | 约束 | 说明 |
|-------|------|------|------|------|
| history_id | BIGINT | - | PK | 历史记录ID（主键） |
| subscription_id | BIGINT | - | NOT NULL | 订购ID |
| user_id | BIGINT | - | NOT NULL | 用户ID |
| product_id | VARCHAR | 50 | NOT NULL | 产品ID |
| operation_type | VARCHAR | 20 | NOT NULL | 操作类型：SUBSCRIBE/UNSUBSCRIBE/SUSPEND/RESUME |
| old_status | VARCHAR | 20 | - | 原状态 |
| new_status | VARCHAR | 20 | NOT NULL | 新状态 |
| operation_time | DATETIME | - | NOT NULL | 操作时间 |
| operation_by | VARCHAR | 50 | NOT NULL | 操作人 |

**索引**：
- PRIMARY KEY (`history_id`)
- INDEX `idx_subscription_id` (`subscription_id`)
- INDEX `idx_user_id` (`user_id`)
- INDEX `idx_operation_time` (`operation_time`)

## 七、ER图

### 7.1 客户服务ER图

```
┌─────────────────────┐
│     customer        │
│─────────────────────│
│ * customer_id (PK)  │
│   customer_type     │
│   status            │
│   level             │
│   points            │
│   version           │
└──────────┬──────────┘
           │
           │ 1:1
           │
    ┌──────┴──────┬──────────────┬──────────────┐
    │             │              │              │
┌───▼────────────┐│          ┌───▼─────────────┐│
│ individual_    ││          │ family_         ││
│ customer_      ││          │ customer_       ││
│ profile        ││          │ profile         ││
│────────────────││          │─────────────────││
│ * customer_id  ││          │ * customer_id   ││
│   name         ││          │   family_name   ││
│   id_number    ││          │   head_member   ││
│   contact_phone││          └──────┬──────────┘│
└────────────────┘│                 │           │
                  │                 │ 1:N       │
                  │          ┌──────▼──────────┐│
                  │          │ family_member   ││
                  │          │─────────────────││
                  │          │ * member_id     ││
                  │          │   customer_id   ││
                  │          │   member_name   ││
                  │          └─────────────────┘│
                  │                             │
           ┌──────▼───────────┐                │
           │ enterprise_      │                │
           │ customer_profile │                │
           │──────────────────│                │
           │ * customer_id    │                │
           │   enterprise_name│                │
           │   org_code       │                │
           │   tax_number     │                │
           └──────────────────┘                │
                                                │
           ┌────────────────────────────────────┘
           │
           │ 1:N
           │
     ┌─────▼──────────────┐
     │ customer_history   │
     │────────────────────│
     │ * history_id       │
     │   customer_id      │
     │   change_type      │
     │   old_value        │
     │   new_value        │
     └────────────────────┘
```

### 7.2 用户服务ER图

```
┌────────────────────┐
│       user         │
│────────────────────│
│ * user_id (PK)     │
│   phone_number (UK)│
│   customer_id      │
│   user_type        │
│   status           │
│   open_time        │
│   active_time      │
│   version          │
└─────────┬──────────┘
          │
          │ 1:1
          │
   ┌──────▼──────────┐
   │   sim_card      │
   │─────────────────│
   │ * sim_card_id   │
   │   user_id       │
   │   iccid (UK)    │
   │   imsi (UK)     │
   │   card_type     │
   │   status        │
   └─────────────────┘
          │
          │ 1:1
          │
   ┌──────▼──────────────┐
   │ service_package     │
   │─────────────────────│
   │ * id                │
   │   user_id           │
   │   package_id        │
   │   package_name      │
   │   monthly_fee       │
   │   effective_time    │
   └─────────────────────┘
          │
          │ 1:N
          │
   ┌──────▼───────────────────┐
   │ package_change_history   │
   │──────────────────────────│
   │ * history_id             │
   │   user_id                │
   │   old_package_id         │
   │   new_package_id         │
   │   change_time            │
   └──────────────────────────┘
          │
          │ 1:N
          │
   ┌──────▼─────────────────┐
   │ user_status_history    │
   │────────────────────────│
   │ * history_id           │
   │   user_id              │
   │   old_status           │
   │   new_status           │
   │   change_time          │
   └────────────────────────┘
```

### 7.3 账户服务ER图

```
┌────────────────────┐
│      account       │
│────────────────────│
│ * account_id (PK)  │
│   customer_id      │
│   account_type     │
│   balance          │
│   frozen_balance   │
│   credit_limit     │
│   status           │
│   version          │
└─────────┬──────────┘
          │
          ├────────────────────┐
          │ 1:N                │ 1:N
          │                    │
   ┌──────▼───────────────┐    │
   │ account_transaction  │    │
   │──────────────────────│    │
   │ * transaction_id     │    │
   │   account_id         │    │
   │   transaction_type   │    │
   │   amount             │    │
   │   balance_before     │    │
   │   balance_after      │    │
   │   transaction_time   │    │
   └──────────────────────┘    │
                               │
                        ┌──────▼─────────────────────┐
                        │ account_user_relationship  │
                        │────────────────────────────│
                        │ * relationship_id          │
                        │   account_id               │
                        │   user_id                  │
                        │   relationship_type        │
                        │   priority                 │
                        │   effective_time           │
                        └────────────────────────────┘
```

### 7.4 产品订购服务ER图

```
┌────────────────────┐
│   subscription     │
│────────────────────│
│ * subscription_id  │
│   user_id          │
│   product_id       │
│   product_name     │
│   subscription_type│
│   status           │
│   effective_time   │
│   version          │
└─────────┬──────────┘
          │
          │ 1:N
          │
   ┌──────▼──────────────────┐
   │ subscription_history    │
   │─────────────────────────│
   │ * history_id            │
   │   subscription_id       │
   │   user_id               │
   │   product_id            │
   │   operation_type        │
   │   old_status            │
   │   new_status            │
   │   operation_time        │
   └─────────────────────────┘
```

## 八、数据库分片策略

### 8.1 分库策略

**按业务垂直拆分：**
1. customer_db - 客户服务数据库
2. user_db - 用户服务数据库
3. account_db - 账户服务数据库
4. subscription_db - 订购服务数据库

### 8.2 分表策略

**用户表（user）：**
- 按user_id取模分256张表
- 预计10亿用户，每表约400万记录
- 分表公式：table_index = user_id % 256

**账户表（account）：**
- 按account_id取模分256张表
- 预计5亿账户，每表约200万记录
- 分表公式：table_index = account_id % 256

**交易表（account_transaction）：**
- 按交易时间按月分表
- 保留最近24个月的数据，历史数据归档
- 表名：account_transaction_YYYYMM

### 8.3 数据冗余策略

为避免跨库查询，适当冗余常用字段：

1. **用户表冗余customer_id**：方便按客户查询用户
2. **套餐表冗余套餐基本信息**：避免频繁查询产商品中心
3. **账户关联表冗余user_id**：方便反向查询

## 九、索引设计

### 9.1 索引设计原则

1. **主键索引**：每张表必须有主键，且使用聚簇索引
2. **唯一索引**：业务唯一字段建立唯一索引（如手机号、证件号）
3. **普通索引**：高频查询字段建立索引
4. **组合索引**：多字段联合查询建立组合索引
5. **覆盖索引**：将查询的所有字段包含在索引中，避免回表

### 9.2 高频查询索引

**客户服务：**
- `idx_id_number` on `id_number` - 按证件号查客户
- `idx_status` on `status` - 按状态查客户

**用户服务：**
- `uk_phone_number` on `phone_number` - 按号码查用户（唯一索引）
- `idx_customer_id` on `customer_id` - 按客户查用户
- `idx_status` on `status` - 按状态查用户

**账户服务：**
- `idx_customer_id` on `customer_id` - 按客户查账户
- `idx_account_id_time` on (`account_id`, `transaction_time`) - 查账户交易记录

## 十、数据一致性保证

### 10.1 单库事务

使用数据库本地事务保证ACID特性。

### 10.2 跨库一致性

**最终一致性（推荐）：**
1. 使用领域事件异步同步数据
2. 使用消息队列保证事件可靠传递
3. 下游服务幂等处理事件

**示例：客户状态变更同步**
1. 客户服务更新客户状态为"停机"
2. 发布CustomerStatusChangedEvent
3. 用户服务订阅事件，更新本地缓存的客户状态

**Saga模式（补偿）：**
对于关键业务流程，使用Saga模式：
1. 每个步骤是一个本地事务
2. 失败时执行补偿事务回滚

## 十一、总结

本ER图设计遵循DDD原则，具有以下特点：

1. **独立数据库**：每个限界上下文独立数据库，松耦合
2. **聚合完整性**：聚合内数据在同一事务中修改
3. **通过ID引用**：聚合间通过ID引用，不用外键
4. **数据冗余**：适当冗余提高查询性能
5. **分库分表**：支持海量数据和高并发
6. **最终一致性**：通过事件驱动保证跨库一致性

这个设计可以支撑：
- 10亿+用户规模
- 300W/天业务办理量
- 4亿次/天服务调用
- 99.9%系统可用性
