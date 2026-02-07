# 电信CRM客户中心 - 领域模型

## 一、核心领域模型概览

本文档详细描述客户中心的领域模型，包括实体（Entity）、值对象（Value Object）、聚合（Aggregate）和领域事件（Domain Event）。

## 二、客户上下文领域模型

### 2.1 聚合：客户聚合（Customer Aggregate）

#### 2.1.1 聚合根：客户（Customer）- 实体

**属性：**
- customerId: CustomerId（值对象） - 客户唯一标识
- customerType: CustomerType（值对象） - 客户类型（个人/家庭/政企）
- profile: CustomerProfile（值对象） - 客户档案
- status: CustomerStatus（值对象） - 客户状态
- level: CustomerLevel（值对象） - 客户等级
- createdTime: DateTime - 创建时间
- updatedTime: DateTime - 更新时间
- version: Long - 版本号（乐观锁）

**行为方法：**
- createCustomer() - 创建客户
- updateProfile(profile: CustomerProfile) - 更新客户资料
- changeStatus(status: CustomerStatus) - 变更客户状态
- upgradeLevel(level: CustomerLevel) - 升级客户等级
- closeAccount() - 销户

**业务不变量：**
- 客户ID全局唯一
- 客户类型一旦确定不可变更
- 销户后不能再进行其他操作
- 客户状态变更必须符合状态机规则

#### 2.1.2 值对象：CustomerProfile（客户档案）

**个人客户档案：**
- name: String - 姓名
- idType: String - 证件类型
- idNumber: String - 证件号码
- gender: String - 性别
- birthDate: Date - 出生日期
- contactPhone: String - 联系电话
- email: String - 电子邮箱
- address: Address（值对象） - 地址

**家庭客户档案：**
- familyName: String - 家庭名称
- headMember: String - 户主姓名
- headIdNumber: String - 户主证件号
- members: List<FamilyMember> - 家庭成员列表
- address: Address - 家庭地址

**政企客户档案：**
- enterpriseName: String - 企业名称
- organizationCode: String - 组织机构代码
- taxNumber: String - 税号
- legalPerson: String - 法人代表
- contactPerson: String - 联系人
- contactPhone: String - 联系电话
- businessAddress: Address - 营业地址

#### 2.1.3 值对象：Address（地址）

**属性：**
- province: String - 省份
- city: String - 城市
- district: String - 区县
- street: String - 街道
- detailAddress: String - 详细地址
- postalCode: String - 邮政编码

#### 2.1.4 值对象：CustomerType（客户类型）

**枚举值：**
- INDIVIDUAL - 个人客户
- FAMILY - 家庭客户
- ENTERPRISE - 政企客户

#### 2.1.5 值对象：CustomerStatus（客户状态）

**枚举值：**
- ACTIVE - 正常
- ARREARS - 欠费
- SUSPENDED - 停机
- CLOSED - 销户

**状态转换规则：**
```
ACTIVE → ARREARS（欠费）
ARREARS → ACTIVE（缴费）
ARREARS → SUSPENDED（长期欠费）
SUSPENDED → ACTIVE（缴清欠费）
SUSPENDED → CLOSED（销户）
ACTIVE → CLOSED（正常销户）
```

#### 2.1.6 值对象：CustomerLevel（客户等级）

**属性：**
- level: Integer - 等级（1-5星）
- points: Integer - 积分
- benefits: List<String> - 权益列表

### 2.2 领域事件

```
CustomerCreatedEvent - 客户创建事件
- customerId: CustomerId
- customerType: CustomerType
- occurredOn: DateTime

CustomerProfileUpdatedEvent - 客户资料更新事件
- customerId: CustomerId
- oldProfile: CustomerProfile
- newProfile: CustomerProfile
- occurredOn: DateTime

CustomerStatusChangedEvent - 客户状态变更事件
- customerId: CustomerId
- oldStatus: CustomerStatus
- newStatus: CustomerStatus
- occurredOn: DateTime

CustomerClosedEvent - 客户销户事件
- customerId: CustomerId
- closeReason: String
- occurredOn: DateTime
```

## 三、用户上下文领域模型

### 3.1 聚合：用户聚合（User Aggregate）

#### 3.1.1 聚合根：用户（User）- 实体

**属性：**
- userId: UserId（值对象） - 用户唯一标识
- phoneNumber: PhoneNumber（值对象） - 手机号码
- customerId: CustomerId（值对象） - 所属客户ID
- userType: UserType（值对象） - 用户类型（个人/群用户）
- status: UserStatus（值对象） - 用户状态
- simCard: SimCard（实体） - SIM卡信息
- servicePackage: ServicePackage（值对象） - 资费套餐
- openTime: DateTime - 开户时间
- activeTime: DateTime - 激活时间
- version: Long - 版本号

**行为方法：**
- openUser(customerId, phoneNumber) - 开户
- activateUser() - 激活
- suspendUser(reason: String) - 停机
- resumeUser() - 复机
- changePackage(newPackage: ServicePackage) - 更改套餐
- replaceSimCard(newSimCard: SimCard) - 换卡
- transferOwnership(newCustomerId: CustomerId) - 过户
- closeUser(reason: String) - 销户

**业务不变量：**
- 用户必须归属于有效客户
- 手机号码全局唯一
- 用户状态变更必须遵循状态机
- 停机用户不能办理除复机外的业务
- 销户用户不能进行任何操作

#### 3.1.2 实体：SimCard（SIM卡）

**属性：**
- simCardId: SimCardId - SIM卡ID
- iccid: String - ICCID（20位）
- imsi: String - IMSI（15位）
- cardType: String - 卡类型（2G/3G/4G/5G）
- status: String - 卡状态（正常/挂失/损坏）
- issueTime: DateTime - 发卡时间

**行为方法：**
- reportLoss() - 挂失
- cancelLoss() - 解挂
- replace() - 换卡

#### 3.1.3 值对象：PhoneNumber（手机号码）

**属性：**
- number: String - 号码（11位）
- areaCode: String - 归属地区号
- operator: String - 运营商（移动/联通/电信）

**验证规则：**
- 必须是11位数字
- 符合号段规则
- 全局唯一

#### 3.1.4 值对象：UserType（用户类型）

**枚举值：**
- INDIVIDUAL - 个人用户
- FAMILY_GROUP - 家庭群用户
- ENTERPRISE_GROUP - 政企群用户

#### 3.1.5 值对象：UserStatus（用户状态）

**枚举值：**
- PRE_ACTIVE - 预开（已开户未激活）
- ACTIVE - 正常
- SUSPENDED_ARREARS - 欠费停机
- SUSPENDED_REPORT - 用户申请停机
- PRE_CLOSE - 预销户
- CLOSED - 已销户

**状态转换规则：**
```
PRE_ACTIVE → ACTIVE（首次激活）
ACTIVE → SUSPENDED_ARREARS（欠费）
ACTIVE → SUSPENDED_REPORT（用户申请）
SUSPENDED_ARREARS → ACTIVE（缴费复机）
SUSPENDED_REPORT → ACTIVE（申请复机）
ACTIVE → PRE_CLOSE（申请销户）
PRE_CLOSE → ACTIVE（取消销户）
PRE_CLOSE → CLOSED（确认销户）
SUSPENDED_* → PRE_CLOSE（申请销户）
```

#### 3.1.6 值对象：ServicePackage（资费套餐）

**属性：**
- packageId: String - 套餐ID
- packageName: String - 套餐名称
- monthlyFee: Money（值对象） - 月租费
- includedTraffic: DataVolume（值对象） - 包含流量
- includedVoice: Integer - 包含语音分钟数
- includedSms: Integer - 包含短信条数
- effectiveRule: EffectiveRule（值对象） - 生效规则

#### 3.1.7 值对象：EffectiveRule（生效规则）

**属性：**
- effectiveType: String - 生效类型（立即/次月）
- effectiveTime: DateTime - 生效时间
- expiryTime: DateTime - 失效时间

### 3.2 领域事件

```
UserOpenedEvent - 用户开户事件
- userId: UserId
- customerId: CustomerId
- phoneNumber: PhoneNumber
- occurredOn: DateTime

UserActivatedEvent - 用户激活事件
- userId: UserId
- activeTime: DateTime
- occurredOn: DateTime

UserStatusChangedEvent - 用户状态变更事件
- userId: UserId
- oldStatus: UserStatus
- newStatus: UserStatus
- reason: String
- occurredOn: DateTime

ServicePackageChangedEvent - 套餐变更事件
- userId: UserId
- oldPackage: ServicePackage
- newPackage: ServicePackage
- effectiveTime: DateTime
- occurredOn: DateTime

SimCardReplacedEvent - 换卡事件
- userId: UserId
- oldSimCard: SimCard
- newSimCard: SimCard
- occurredOn: DateTime

UserTransferredEvent - 用户过户事件
- userId: UserId
- oldCustomerId: CustomerId
- newCustomerId: CustomerId
- occurredOn: DateTime

UserClosedEvent - 用户销户事件
- userId: UserId
- closeReason: String
- occurredOn: DateTime
```

## 四、账户上下文领域模型

### 4.1 聚合：账户聚合（Account Aggregate）

#### 4.1.1 聚合根：账户（Account）- 实体

**属性：**
- accountId: AccountId（值对象） - 账户ID
- customerId: CustomerId（值对象） - 所属客户ID
- accountType: AccountType（值对象） - 账户类型
- balance: Money（值对象） - 账户余额
- status: AccountStatus（值对象） - 账户状态
- creditLimit: Money（值对象） - 信用额度
- openTime: DateTime - 开户时间
- version: Long - 版本号

**行为方法：**
- openAccount(customerId: CustomerId) - 开户
- recharge(amount: Money, channel: String) - 充值
- deduct(amount: Money, reason: String) - 扣费
- freeze(amount: Money) - 冻结
- unfreeze(amount: Money) - 解冻
- closeAccount() - 销户

**业务不变量：**
- 账户余额不能为负（除非有信用额度）
- 余额 = 可用余额 + 冻结金额
- 扣费时检查可用余额是否充足
- 账户必须归属于有效客户

#### 4.1.2 实体：AccountTransaction（账户交易）

**属性：**
- transactionId: String - 交易流水号
- accountId: AccountId - 账户ID
- transactionType: TransactionType - 交易类型
- amount: Money - 交易金额
- balanceBefore: Money - 交易前余额
- balanceAfter: Money - 交易后余额
- transactionTime: DateTime - 交易时间
- channel: String - 交易渠道
- description: String - 交易描述

#### 4.1.3 值对象：Money（金额）

**属性：**
- amount: BigDecimal - 金额
- currency: String - 货币单位（CNY）

**方法：**
- add(other: Money): Money
- subtract(other: Money): Money
- multiply(factor: BigDecimal): Money
- isPositive(): Boolean
- isZero(): Boolean

#### 4.1.4 值对象：AccountType（账户类型）

**枚举值：**
- PREPAID - 预付费账户
- POSTPAID - 后付费账户
- MIXED - 混合账户

#### 4.1.5 值对象：AccountStatus（账户状态）

**枚举值：**
- ACTIVE - 正常
- FROZEN - 冻结
- CLOSED - 已关闭

#### 4.1.6 值对象：TransactionType（交易类型）

**枚举值：**
- RECHARGE - 充值
- DEDUCTION - 扣费
- REFUND - 退款
- FREEZE - 冻结
- UNFREEZE - 解冻
- TRANSFER - 转账

### 4.2 聚合：账户关联聚合（Account Relationship Aggregate）

#### 4.2.1 聚合根：账户关联（AccountRelationship）- 实体

**属性：**
- relationshipId: String - 关联ID
- accountId: AccountId - 账户ID
- userId: UserId - 用户ID
- relationshipType: String - 关联类型（主关联/副关联）
- priority: Integer - 优先级（扣费顺序）
- effectiveTime: DateTime - 生效时间
- expiryTime: DateTime - 失效时间

**行为方法：**
- bindUser(userId: UserId) - 绑定用户
- unbindUser(userId: UserId) - 解绑用户
- changePriority(newPriority: Integer) - 调整优先级

### 4.3 领域事件

```
AccountOpenedEvent - 账户开户事件
- accountId: AccountId
- customerId: CustomerId
- accountType: AccountType
- occurredOn: DateTime

AccountRechargedEvent - 账户充值事件
- accountId: AccountId
- amount: Money
- channel: String
- balanceAfter: Money
- occurredOn: DateTime

AccountDeductedEvent - 账户扣费事件
- accountId: AccountId
- amount: Money
- reason: String
- balanceAfter: Money
- occurredOn: DateTime

AccountBalanceInsufficientEvent - 余额不足事件
- accountId: AccountId
- requiredAmount: Money
- currentBalance: Money
- occurredOn: DateTime

AccountFrozenEvent - 账户冻结事件
- accountId: AccountId
- freezeAmount: Money
- occurredOn: DateTime

AccountClosedEvent - 账户关闭事件
- accountId: AccountId
- closeReason: String
- occurredOn: DateTime

UserAccountBoundEvent - 用户账户绑定事件
- accountId: AccountId
- userId: UserId
- occurredOn: DateTime
```

## 五、产品订购上下文领域模型

### 5.1 聚合：产品订购聚合（Subscription Aggregate）

#### 5.1.1 聚合根：产品订购（Subscription）- 实体

**属性：**
- subscriptionId: SubscriptionId - 订购ID
- userId: UserId - 用户ID
- productId: ProductId - 产品ID
- productName: String - 产品名称
- subscriptionType: SubscriptionType - 订购类型
- status: SubscriptionStatus - 订购状态
- effectiveTime: DateTime - 生效时间
- expiryTime: DateTime - 失效时间
- subscribeTime: DateTime - 订购时间
- version: Long - 版本号

**行为方法：**
- subscribe(userId, productId) - 订购
- unsubscribe() - 退订
- suspend() - 暂停
- resume() - 恢复
- extend(period: Integer) - 续期

**业务不变量：**
- 订购必须基于有效用户
- 同一产品不能重复订购（除非允许多实例）
- 互斥产品不能同时订购
- 依赖产品必须先订购

#### 5.1.2 值对象：SubscriptionType（订购类型）

**枚举值：**
- PACKAGE - 套餐类
- VALUE_ADDED - 增值业务类
- DISCOUNT - 优惠类

#### 5.1.3 值对象：SubscriptionStatus（订购状态）

**枚举值：**
- PENDING - 待生效
- ACTIVE - 生效中
- SUSPENDED - 已暂停
- EXPIRED - 已失效
- CANCELLED - 已取消

### 5.2 领域事件

```
ProductSubscribedEvent - 产品订购事件
- subscriptionId: SubscriptionId
- userId: UserId
- productId: ProductId
- occurredOn: DateTime

ProductUnsubscribedEvent - 产品退订事件
- subscriptionId: SubscriptionId
- userId: UserId
- productId: ProductId
- occurredOn: DateTime

SubscriptionStatusChangedEvent - 订购状态变更事件
- subscriptionId: SubscriptionId
- oldStatus: SubscriptionStatus
- newStatus: SubscriptionStatus
- occurredOn: DateTime
```

## 六、领域模型关系图

```
┌──────────────────────────────────────────────────────────────┐
│                    客户上下文                                  │
│  ┌────────────────────────────────────────────────────┐      │
│  │  Customer Aggregate (客户聚合)                      │      │
│  │  ┌──────────────────────────┐                      │      │
│  │  │  Customer (聚合根)        │                      │      │
│  │  │  - customerId            │◄───────┐             │      │
│  │  │  - customerType          │        │             │      │
│  │  │  - profile               │        │             │      │
│  │  │  - status                │        │             │      │
│  │  │  - level                 │        │             │      │
│  │  └──────────────────────────┘        │             │      │
│  └────────────────────────────────────────┘           │      │
└────────────────────────────┬──────────────────────────┘      │
                             │ 引用                             │
                             │                                  │
┌────────────────────────────▼──────────────────────────┐      │
│                    用户上下文                          │      │
│  ┌────────────────────────────────────────────────┐   │      │
│  │  User Aggregate (用户聚合)                      │   │      │
│  │  ┌──────────────────────────┐                  │   │      │
│  │  │  User (聚合根)            │                  │   │      │
│  │  │  - userId                │                  │   │      │
│  │  │  - customerId ───────────┼──────────────────┘   │      │
│  │  │  - phoneNumber           │                      │      │
│  │  │  - userType              │                      │      │
│  │  │  - status                │                      │      │
│  │  │  - simCard               │◄────┐                │      │
│  │  │  - servicePackage        │     │                │      │
│  │  └──────────────────────────┘     │                │      │
│  │        │ 包含                      │                │      │
│  │  ┌─────▼──────────────────┐       │                │      │
│  │  │  SimCard (实体)        │       │                │      │
│  │  │  - simCardId           │       │                │      │
│  │  │  - iccid               │       │                │      │
│  │  │  - imsi                │       │                │      │
│  │  └────────────────────────┘       │                │      │
│  └────────────────────────────────────┘               │      │
└────────────────────────────┬──────────────────────────┘      │
                             │ 引用                             │
                             │                                  │
┌────────────────────────────▼──────────────────────────┐      │
│                    账户上下文                          │      │
│  ┌────────────────────────────────────────────────┐   │      │
│  │  Account Aggregate (账户聚合)                   │   │      │
│  │  ┌──────────────────────────┐                  │   │      │
│  │  │  Account (聚合根)         │                  │   │      │
│  │  │  - accountId             │                  │   │      │
│  │  │  - customerId            │                  │   │      │
│  │  │  - balance               │                  │   │      │
│  │  │  - status                │                  │   │      │
│  │  └──────────────────────────┘                  │   │      │
│  │        │ 1:N                                    │   │      │
│  │  ┌─────▼──────────────────┐                    │   │      │
│  │  │ AccountTransaction     │                    │   │      │
│  │  │  - transactionId       │                    │   │      │
│  │  │  - amount              │                    │   │      │
│  │  │  - transactionType     │                    │   │      │
│  │  └────────────────────────┘                    │   │      │
│  └────────────────────────────────────────────────┘   │      │
│                                                        │      │
│  ┌────────────────────────────────────────────────┐   │      │
│  │  AccountRelationship Aggregate                 │   │      │
│  │  ┌──────────────────────────┐                  │   │      │
│  │  │ AccountRelationship      │                  │   │      │
│  │  │  - accountId             │                  │   │      │
│  │  │  - userId                │                  │   │      │
│  │  │  - priority              │                  │   │      │
│  │  └──────────────────────────┘                  │   │      │
│  └────────────────────────────────────────────────┘   │      │
└────────────────────────────────────────────────────────┘      │
                                                                 │
┌────────────────────────────────────────────────────────────┐  │
│                    产品订购上下文                           │  │
│  ┌────────────────────────────────────────────────┐       │  │
│  │  Subscription Aggregate (订购聚合)              │       │  │
│  │  ┌──────────────────────────┐                  │       │  │
│  │  │ Subscription (聚合根)     │                  │       │  │
│  │  │  - subscriptionId        │                  │       │  │
│  │  │  - userId                │                  │       │  │
│  │  │  - productId             │                  │       │  │
│  │  │  - status                │                  │       │  │
│  │  └──────────────────────────┘                  │       │  │
│  └────────────────────────────────────────────────┘       │  │
└────────────────────────────────────────────────────────────┘  │
```

## 七、设计要点总结

### 7.1 聚合设计原则
1. **小聚合优先**：每个聚合保持精简，避免大聚合
2. **通过ID引用其他聚合**：不在聚合内部直接包含其他聚合
3. **最终一致性**：聚合之间通过领域事件保持最终一致性
4. **事务边界**：一次事务只修改一个聚合

### 7.2 实体与值对象区分
- **实体**：有唯一标识，生命周期独立（Customer, User, Account）
- **值对象**：无唯一标识，可替换，不可变（Money, Address, PhoneNumber）

### 7.3 领域事件使用
- 用于聚合间的解耦通信
- 记录重要的业务状态变更
- 支持事件溯源和审计
- 触发下游业务流程

### 7.4 业务规则封装
- 在聚合根中封装核心业务规则
- 通过领域服务处理跨聚合的业务逻辑
- 使用规格模式（Specification）处理复杂查询条件
