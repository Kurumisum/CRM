# 电信CRM客户中心 - 微服务拆分清单

## 一、概述

基于DDD领域驱动设计方法论，遵循限界上下文原则，将客户中心拆分为多个独立的微服务。每个微服务负责一个明确的业务领域，具有独立的数据库和部署单元。

## 二、微服务拆分原则

### 2.1 DDD拆分原则

1. **限界上下文对应微服务**：每个限界上下文对应一个或多个微服务
2. **高内聚低耦合**：微服务内部高度内聚，服务间松耦合
3. **独立数据库**：每个微服务拥有独立的数据库
4. **业务能力对齐**：按业务能力而非技术层次拆分

### 2.2 微服务立方体模型

按照微服务立方体（Microservices Cube）三个维度考虑拆分：

**X轴（水平复制）：**
- 通过负载均衡将请求分发到多个服务实例
- 提高可用性和吞吐量

**Y轴（功能分解）：**
- 按业务功能拆分为不同的微服务
- 本文档重点关注Y轴拆分

**Z轴（数据分片）：**
- 按数据范围（如客户ID、地域）拆分
- 提高数据处理能力

## 三、核心微服务清单

### 3.1 客户管理服务（Customer Service）

#### 3.1.1 服务定位
- **英文名称**：customer-service
- **端口**：8081
- **数据库**：customer_db
- **所属上下文**：客户上下文（Customer Context）

#### 3.1.2 核心职责
1. **客户生命周期管理**
   - 客户开户（个人、家庭、政企）
   - 客户资料管理与变更
   - 客户销户处理
   
2. **客户分类与等级管理**
   - 客户类型管理（个人/家庭/政企）
   - 客户等级评定与升降级
   - 客户积分管理

3. **客户关系管理**
   - 家庭成员关系维护
   - 政企组织架构管理
   - 客户关联关系查询

4. **实名认证**
   - 集成第三方实名认证系统
   - 身份证验证
   - 一证五户限制检查

#### 3.1.3 边界划分依据
- **领域概念**：客户是独立的领域概念，代表合同签约方
- **生命周期**：客户有独立的生命周期，与用户不同步
- **业务规则**：客户层面的业务规则（如客户等级、一证五户）
- **数据隔离**：客户数据敏感，需要独立管理

#### 3.1.4 对外接口
- `POST /api/customers` - 创建客户
- `GET /api/customers/{customerId}` - 查询客户信息
- `PUT /api/customers/{customerId}` - 更新客户资料
- `DELETE /api/customers/{customerId}` - 销户
- `GET /api/customers/{customerId}/level` - 查询客户等级
- `PUT /api/customers/{customerId}/level` - 升级客户等级

#### 3.1.5 发布事件
- `CustomerCreatedEvent` - 客户创建
- `CustomerProfileUpdatedEvent` - 客户资料更新
- `CustomerStatusChangedEvent` - 客户状态变更
- `CustomerLevelUpgradedEvent` - 客户等级升级
- `CustomerClosedEvent` - 客户销户

#### 3.1.6 技术栈
- Spring Boot 3.x
- Spring Data JPA
- MySQL 8.0
- Redis（缓存）
- Kafka（事件发布）

---

### 3.2 用户管理服务（User Service）

#### 3.2.1 服务定位
- **英文名称**：user-service
- **端口**：8082
- **数据库**：user_db
- **所属上下文**：用户上下文（User Context）

#### 3.2.2 核心职责
1. **用户生命周期管理**
   - 个人用户开户（高频，占90%）
   - 群用户开户
   - 用户激活
   - 用户停机/复机
   - 用户销户

2. **用户状态管理**
   - 用户状态机维护
   - 欠费停机自动处理
   - 用户状态变更历史

3. **资费套餐管理**
   - 套餐变更（立即/次月生效）
   - 套餐生效规则处理
   - 套餐变更历史

4. **SIM卡管理**
   - SIM卡绑定与更换
   - SIM卡挂失/解挂
   - SIM卡状态管理

5. **过户业务**
   - 用户过户（归属客户变更）
   - 过户权限验证
   - 过户记录管理

#### 3.2.3 边界划分依据
- **高频业务**：用户管理占业务量90%以上，需要独立优化
- **状态复杂**：用户状态机复杂，有6个状态和多个转换规则
- **性能要求**：需要支持高并发，独立扩展
- **领域概念清晰**：用户（号码）是独立的领域概念

#### 3.2.4 对外接口
- `POST /api/users` - 创建用户
- `GET /api/users/{userId}` - 查询用户信息
- `GET /api/users?phoneNumber={phoneNumber}` - 按号码查询
- `GET /api/users?customerId={customerId}` - 按客户查询用户列表
- `POST /api/users/{userId}/activate` - 激活用户
- `POST /api/users/{userId}/suspend` - 停机
- `POST /api/users/{userId}/resume` - 复机
- `POST /api/users/{userId}/change-package` - 变更套餐
- `POST /api/users/{userId}/replace-sim` - 换卡
- `POST /api/users/{userId}/transfer` - 过户
- `DELETE /api/users/{userId}` - 销户

#### 3.2.5 发布事件
- `UserOpenedEvent` - 用户开户
- `UserActivatedEvent` - 用户激活
- `UserStatusChangedEvent` - 用户状态变更
- `ServicePackageChangedEvent` - 套餐变更
- `SimCardReplacedEvent` - 换卡
- `UserTransferredEvent` - 用户过户
- `UserClosedEvent` - 用户销户

#### 3.2.6 技术栈
- Spring Boot 3.x
- Spring Data JPA
- MySQL 8.0（分库分表）
- Redis（缓存、分布式锁）
- Kafka（事件发布）
- ShardingSphere（分库分表）

---

### 3.3 账户管理服务（Account Service）

#### 3.3.1 服务定位
- **英文名称**：account-service
- **端口**：8083
- **数据库**：account_db
- **所属上下文**：账户上下文（Account Context）

#### 3.3.2 核心职责
1. **账户生命周期管理**
   - 账户开立（预付费/后付费）
   - 账户关闭
   - 账户状态管理

2. **账户余额管理**
   - 账户充值
   - 账户扣费
   - 账户余额查询
   - 账户余额不足处理

3. **账户冻结与解冻**
   - 账户冻结（司法/风控）
   - 账户解冻
   - 冻结金额管理

4. **账户关联关系管理**
   - 账户-用户绑定
   - 账户-用户解绑
   - 扣费优先级管理

5. **账户交易管理**
   - 交易记录查询
   - 交易对账
   - 交易统计

#### 3.3.3 边界划分依据
- **资金安全**：涉及资金，需要独立的安全边界和审计
- **计费解耦**：作为计费的抽象层，隔离复杂的计费逻辑
- **灵活关联**：支持多种账户-用户关联模式
- **独立演进**：可能需要接入支付平台等外部系统

#### 3.3.4 对外接口
- `POST /api/accounts` - 创建账户
- `GET /api/accounts/{accountId}` - 查询账户信息
- `GET /api/accounts?customerId={customerId}` - 按客户查询账户
- `POST /api/accounts/{accountId}/recharge` - 充值
- `POST /api/accounts/{accountId}/deduct` - 扣费
- `POST /api/accounts/{accountId}/freeze` - 冻结
- `POST /api/accounts/{accountId}/unfreeze` - 解冻
- `POST /api/accounts/{accountId}/bind-user` - 绑定用户
- `DELETE /api/accounts/{accountId}/unbind-user` - 解绑用户
- `GET /api/accounts/{accountId}/transactions` - 查询交易记录
- `DELETE /api/accounts/{accountId}` - 关闭账户

#### 3.3.5 发布事件
- `AccountOpenedEvent` - 账户开立
- `AccountRechargedEvent` - 账户充值
- `AccountDeductedEvent` - 账户扣费
- `AccountBalanceInsufficientEvent` - 余额不足
- `AccountFrozenEvent` - 账户冻结
- `AccountUnfrozenEvent` - 账户解冻
- `UserAccountBoundEvent` - 用户账户绑定
- `AccountClosedEvent` - 账户关闭

#### 3.3.6 技术栈
- Spring Boot 3.x
- Spring Data JPA
- MySQL 8.0（分库分表）
- Redis（缓存）
- Kafka（事件发布）
- ShardingSphere（分库分表）

---

### 3.4 产品订购服务（Subscription Service）

#### 3.4.1 服务定位
- **英文名称**：subscription-service
- **端口**：8084
- **数据库**：subscription_db
- **所属上下文**：产品订购上下文（Subscription Context）

#### 3.4.2 核心职责
1. **产品订购管理**
   - 产品订购（套餐、增值业务）
   - 产品退订
   - 产品变更

2. **订购关系管理**
   - 订购关系查询
   - 用户已订购产品列表
   - 订购历史查询

3. **订购生效规则处理**
   - 立即生效
   - 次月生效
   - 指定日期生效
   - 定时任务处理待生效订购

4. **产品规则检查**
   - 产品互斥关系检查
   - 产品依赖关系检查
   - 订购资格检查

#### 3.4.3 边界划分依据
- **业务独立**：产品订购是独立的业务流程
- **规则复杂**：产品间有复杂的互斥和依赖关系
- **独立演进**：产品订购规则频繁变化，需要独立演进
- **与产商品中心协同**：需要频繁查询产品信息

#### 3.4.4 对外接口
- `POST /api/subscriptions` - 订购产品
- `GET /api/subscriptions/{subscriptionId}` - 查询订购详情
- `GET /api/subscriptions?userId={userId}` - 查询用户订购列表
- `DELETE /api/subscriptions/{subscriptionId}` - 退订产品
- `POST /api/subscriptions/{subscriptionId}/suspend` - 暂停订购
- `POST /api/subscriptions/{subscriptionId}/resume` - 恢复订购
- `GET /api/subscriptions/{subscriptionId}/history` - 查询订购历史

#### 3.4.5 发布事件
- `ProductSubscribedEvent` - 产品订购
- `ProductUnsubscribedEvent` - 产品退订
- `SubscriptionStatusChangedEvent` - 订购状态变更
- `SubscriptionActivatedEvent` - 订购生效

#### 3.4.6 技术栈
- Spring Boot 3.x
- Spring Data JPA
- MySQL 8.0
- Redis（缓存）
- Kafka（事件发布）
- Scheduled Tasks（定时任务）

---

## 四、基础设施服务

### 4.1 API网关服务（API Gateway）

#### 4.1.1 服务定位
- **英文名称**：api-gateway
- **端口**：8080
- **类型**：边缘服务

#### 4.1.2 核心职责
1. **请求路由**
   - 根据URL路径路由到对应的微服务
   - 支持动态路由配置

2. **统一认证授权**
   - JWT Token验证
   - 权限检查
   - 会话管理

3. **流量控制**
   - 限流（IP限流、用户限流、接口限流）
   - 熔断
   - 降级

4. **协议转换**
   - HTTP -> gRPC
   - REST -> GraphQL

5. **日志与监控**
   - 请求日志记录
   - 性能监控
   - 调用链追踪

#### 4.1.3 技术栈
- Spring Cloud Gateway
- Spring Security
- Redis（限流、会话）
- Sentinel（流控、熔断）

---

### 4.2 配置中心（Config Service）

#### 4.2.1 服务定位
- **英文名称**：config-service
- **端口**：8888
- **类型**：基础设施服务

#### 4.2.2 核心职责
1. **集中配置管理**
   - 配置统一存储
   - 配置版本管理
   - 配置回滚

2. **动态配置推送**
   - 配置变更实时推送
   - 不重启更新配置

3. **环境隔离**
   - 开发环境配置
   - 测试环境配置
   - 生产环境配置

#### 4.2.3 技术栈
- Spring Cloud Config
- Git（配置存储）
- Spring Cloud Bus（配置推送）

---

### 4.3 服务注册中心（Registry Service）

#### 4.3.1 服务定位
- **英文名称**：registry-service
- **端口**：8761
- **类型**：基础设施服务

#### 4.3.2 核心职责
1. **服务注册**
   - 服务实例注册
   - 服务实例下线

2. **服务发现**
   - 服务地址查询
   - 服务健康检查

3. **负载均衡**
   - 客户端负载均衡
   - 支持多种负载均衡策略

#### 4.3.3 技术栈
- Nacos / Consul / Eureka
- Ribbon（客户端负载均衡）

---

### 4.4 消息服务（Message Service）

#### 4.4.1 服务定位
- **英文名称**：message-service
- **端口**：8085
- **数据库**：message_db
- **类型**：支撑服务

#### 4.4.2 核心职责
1. **短信发送**
   - 验证码短信
   - 通知短信
   - 营销短信

2. **邮件发送**
   - 通知邮件
   - 账单邮件

3. **站内消息**
   - 消息推送
   - 消息查询

4. **消息模板管理**
   - 模板创建
   - 模板变量替换

#### 4.4.3 技术栈
- Spring Boot
- 阿里云短信服务
- 腾讯云短信服务
- JavaMail

---

## 五、微服务交互关系

### 5.1 服务调用关系图

```
┌──────────────────────────────────────────────────────────┐
│                     API Gateway                          │
│                   (8080)                                  │
└────────┬──────────────┬──────────────┬──────────────┬────┘
         │              │              │              │
         │              │              │              │
    ┌────▼────┐    ┌───▼────┐    ┌───▼────┐    ┌───▼────────┐
    │Customer │    │  User  │    │Account │    │Subscription│
    │Service  │    │Service │    │Service │    │  Service   │
    │ (8081)  │    │ (8082) │    │ (8083) │    │  (8084)    │
    └────┬────┘    └───┬────┘    └───┬────┘    └───┬────────┘
         │             │              │              │
         │             │              │              │
         └─────────────┴──────────────┴──────────────┘
                       │
                       │ Events (Kafka)
                       │
         ┌─────────────▼──────────────┐
         │      Event Bus (Kafka)     │
         └────────────────────────────┘
                       │
                       │ Subscribe Events
                       │
         ┌─────────────▼──────────────┐
         │   Message Service (8085)   │
         │   (发送通知短信/邮件)       │
         └────────────────────────────┘
```

### 5.2 服务依赖关系

**客户服务 (Customer Service):**
- 依赖：无直接依赖
- 被依赖：用户服务、账户服务

**用户服务 (User Service):**
- 依赖：客户服务（验证客户存在）
- 被依赖：账户服务、订购服务

**账户服务 (Account Service):**
- 依赖：客户服务（验证客户）、用户服务（验证用户）
- 被依赖：无

**订购服务 (Subscription Service):**
- 依赖：用户服务（验证用户）
- 被依赖：无

### 5.3 同步调用vs异步事件

**使用同步调用（RESTful API）：**
1. 强一致性要求的场景
2. 需要立即返回结果的场景
3. 查询操作

**使用异步事件（Kafka）：**
1. 跨服务通知
2. 最终一致性场景
3. 状态变更通知

## 六、微服务规模估算

### 6.1 服务实例数量

基于每日300W业务量、4亿次服务调用的需求：

| 服务 | 平峰期实例数 | 高峰期实例数 | 说明 |
|------|------------|------------|------|
| API Gateway | 4 | 8 | 统一入口，高流量 |
| Customer Service | 2 | 4 | 低频操作 |
| User Service | 8 | 16 | 高频操作，占90% |
| Account Service | 4 | 8 | 充值扣费高频 |
| Subscription Service | 2 | 4 | 中等频率 |
| Message Service | 2 | 4 | 异步处理 |

**总计：22个实例（平峰期）→ 44个实例（高峰期）**

### 6.2 资源配置

**每个服务实例配置：**
- CPU: 2核
- 内存: 4GB
- 磁盘: 50GB

**数据库配置：**
- 主库：8核32GB
- 从库：8核32GB（2个）

**Redis集群：**
- 3主3从，每个节点：4核16GB

**Kafka集群：**
- 3个Broker，每个：4核16GB

## 七、微服务拆分的优势

### 7.1 技术优势
1. **独立部署**：每个服务可以独立部署，不影响其他服务
2. **技术栈自由**：不同服务可以选择不同的技术栈
3. **故障隔离**：一个服务故障不会影响其他服务
4. **水平扩展**：可以针对高负载服务独立扩展

### 7.2 业务优势
1. **团队自治**：每个团队负责一个或几个微服务
2. **快速迭代**：服务间松耦合，可以快速迭代
3. **业务聚焦**：每个服务聚焦单一业务领域
4. **灵活变更**：业务变更影响范围小

### 7.3 运维优势
1. **容易监控**：每个服务独立监控
2. **问题定位**：故障定位更精准
3. **灰度发布**：可以对单个服务进行灰度发布
4. **资源优化**：可以针对不同服务优化资源配置

## 八、微服务拆分的挑战

### 8.1 分布式事务
- **挑战**：跨服务的事务一致性难以保证
- **解决方案**：使用Saga模式、事件溯源、最终一致性

### 8.2 服务调用复杂度
- **挑战**：服务间调用链路复杂，难以追踪
- **解决方案**：使用分布式追踪（如Zipkin、Skywalking）

### 8.3 数据一致性
- **挑战**：数据分散在多个数据库，难以保证一致性
- **解决方案**：通过事件驱动架构，保证最终一致性

### 8.4 运维复杂度
- **挑战**：服务数量多，运维复杂度高
- **解决方案**：使用容器化（Docker）、编排工具（Kubernetes）、自动化运维

## 九、总结

本微服务拆分方案基于DDD方法论，将客户中心拆分为4个核心微服务：

1. **客户管理服务**：管理客户生命周期和档案
2. **用户管理服务**：管理用户（号码）全生命周期（高频、核心）
3. **账户管理服务**：管理账户余额和交易
4. **产品订购服务**：管理产品订购关系

加上4个基础设施服务：

5. **API网关服务**：统一入口、认证授权、流控
6. **配置中心**：集中配置管理
7. **服务注册中心**：服务注册与发现
8. **消息服务**：短信、邮件、站内消息

**拆分特点：**
- **清晰边界**：每个服务有明确的职责边界
- **独立数据库**：每个服务拥有独立的数据库
- **事件驱动**：通过事件实现服务间解耦
- **可独立扩展**：针对高负载服务独立扩展
- **业务对齐**：微服务与业务能力对齐

这个拆分方案能够支撑每日300W业务量、4亿次服务调用，满足高并发、高可用的要求。
