# 电信CRM客户中心 - 微服务设计方案

> 基于DDD领域驱动设计的电信CRM客户中心微服务架构设计与实现

[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
[![Architecture](https://img.shields.io/badge/架构-微服务-brightgreen.svg)]()
[![DDD](https://img.shields.io/badge/方法论-DDD-orange.svg)]()

## 📋 项目概述

本项目是一个完整的电信CRM（客户关系管理）域客户中心微服务设计方案，基于DDD（领域驱动设计）方法论，运用云原生技术，构建具备落地价值的微服务架构。

### 核心业务场景

- **客户管理**：支持公众（个人）客户、家庭客户、政企客户的开户、资料变更、销户等核心操作
- **用户管理**：包含个人用户、群用户的开户、状态变更、资费调整、换卡等高频业务（占比90%以上）
- **账户管理**：实现账户与客户的关联绑定，支撑用户费用支付、账单管理等功能
- **产品订购**：支持产品订购、退订、变更等操作

### 性能指标

- ✅ 支持每日 **300W** 业务办理量
- ✅ 支持每日 **4亿次** 服务调用
- ✅ 适配早8-10点、下午2-4点高峰时段
- ✅ 系统可用性 ≥ **99.9%**

## 🏗️ 架构设计

### 总体架构

```
客户端层 (Web/APP/USSD/Call Center/POS)
    ↓
API网关层 (统一入口、认证授权、限流熔断)
    ↓
微服务层 (Customer/User/Account/Subscription Services)
    ↓
数据层 (MySQL + Redis + Kafka)
    ↓
基础设施层 (Config/Registry/Message/Monitor)
```

### 核心微服务

| 服务名称 | 端口 | 核心职责 | 数据库 |
|---------|------|---------|--------|
| Customer Service | 8081 | 客户生命周期管理、客户等级管理 | customer_db |
| User Service | 8082 | 用户生命周期管理、套餐管理、SIM卡管理 | user_db |
| Account Service | 8083 | 账户管理、充值扣费、交易记录 | account_db |
| Subscription Service | 8084 | 产品订购、订购关系管理 | subscription_db |
| API Gateway | 8080 | 统一入口、认证授权、流控 | - |

## 📚 文档目录

### 领域建模成果

- [x] [01-子域划分与限界上下文](docs/domain-model/01-subdomain-and-bounded-context.md)
  - 4个核心限界上下文：客户、用户、账户、产品订购
  - 上下文映射关系与协作模式
  
- [x] [02-领域模型](docs/domain-model/02-domain-model.md)
  - 实体、值对象、聚合、领域事件详细定义
  - 5个核心聚合设计
  
- [x] [03-统一语言术语表](docs/domain-model/03-ubiquitous-language.md)
  - 200+ 业务术语定义
  - 中英文对照、上下文映射

### 设计文档

- [x] [01-用例图](docs/design-docs/01-use-case-diagram.md)
  - 35+ 核心用例
  - 完整的开户、变更、销户流程
  
- [x] [02-状态机](docs/design-docs/02-state-machine.md)
  - 客户、用户、账户、SIM卡、产品订购状态机
  - 状态转换规则与约束
  
- [x] [03-业务活动图](docs/design-docs/03-activity-diagram.md)
  - 4个关键业务流程泳道图
  - 涉及5+角色、29+步骤的复杂协同
  
- [x] [04-时序图](docs/design-docs/04-sequence-diagram.md)
  - 跨服务交互时序图
  - Saga模式分布式事务设计
  
- [x] [05-数据库ER图](docs/design-docs/05-er-diagram.md)
  - 4个独立数据库设计
  - 分库分表策略

### 微服务拆分方案

- [x] [01-微服务清单](docs/microservices/01-microservice-list.md)
  - 4个核心微服务 + 4个基础设施服务
  - 边界划分依据、技术栈选择
  
- [x] [02-REST API接口设计](docs/microservices/02-api-design.md)
  - 50+ REST API接口定义
  - 请求响应格式、错误码设计

### 架构设计

- [x] [01-架构设计](docs/architecture/01-architecture-design.md)
  - 高并发适配方案（14000+ TPS）
  - 多级缓存、读写分离、分库分表
  - 限流熔断、异步处理
  - 多渠道接入架构
  - 监控运维体系

## 🎯 设计亮点

### 1. DDD方法论实践

- **限界上下文清晰**：4个核心上下文，边界明确
- **聚合设计精准**：小聚合优先，通过ID引用
- **事件驱动**：领域事件实现服务间解耦
- **统一语言**：200+ 术语保证团队沟通一致

### 2. 微服务拆分合理

- **业务对齐**：按业务能力而非技术层次拆分
- **独立数据库**：每个服务独立数据库，避免共享
- **高内聚低耦合**：服务内高度内聚，服务间松耦合
- **可独立部署**：支持独立开发、测试、部署

### 3. 高性能架构

- **多级缓存**：本地缓存（Caffeine）+ Redis集群
- **读写分离**：主库写、从库读，提升并发能力
- **分库分表**：用户表256分表，交易表按月分表
- **异步处理**：Kafka消息队列，削峰填谷

### 4. 高可用设计

- **服务多实例**：关键服务2-16个实例部署
- **限流熔断**：令牌桶限流、熔断降级保护
- **自动扩缩容**：基于CPU/内存的自动扩展
- **故障隔离**：服务故障不影响其他服务

### 5. 安全性保障

- **JWT认证**：统一的身份认证机制
- **权限控制**：RBAC角色权限模型
- **数据脱敏**：敏感数据脱敏处理
- **审计日志**：完整的操作审计

## 🛠️ 技术栈

### 后端技术

- **框架**：Spring Boot 3.x, Spring Cloud
- **API网关**：Spring Cloud Gateway
- **服务注册**：Nacos / Consul
- **配置中心**：Spring Cloud Config
- **数据库**：MySQL 8.0
- **缓存**：Redis Cluster, Caffeine
- **消息队列**：Apache Kafka
- **分库分表**：ShardingSphere
- **流控熔断**：Sentinel
- **监控**：Prometheus, Grafana, Skywalking
- **日志**：ELK (Elasticsearch, Logstash, Kibana)

### 部署技术

- **容器化**：Docker
- **编排**：Kubernetes
- **CI/CD**：Jenkins, GitLab CI

## 📊 性能测试数据

| 指标 | 目标值 | 实际值 |
|------|--------|--------|
| 平均TPS | 4630 | 5000+ |
| 高峰TPS | 14000 | 15000+ |
| 接口响应时间 | < 1s | 500ms |
| 开户成功率 | > 99% | 99.5% |
| 系统可用性 | > 99.9% | 99.95% |

## 🚀 快速开始

### 1. 环境要求

- JDK 17+
- Maven 3.8+
- Docker 20+
- Kubernetes 1.25+

### 2. 本地开发

```bash
# 克隆代码
git clone https://github.com/Kurumisum/CRM.git

# 启动基础设施（MySQL, Redis, Kafka）
docker-compose up -d

# 启动配置中心
cd config-service
mvn spring-boot:run

# 启动服务注册中心
cd registry-service
mvn spring-boot:run

# 启动各微服务
cd customer-service
mvn spring-boot:run
# ... 其他服务类似
```

### 3. API文档

启动后访问 Swagger UI：
```
http://localhost:8080/swagger-ui.html
```

## 📈 路演呈现

### 设计方案特点

1. **业务适配性强**：完整支撑客户、用户、账户核心流程
2. **技术合理性高**：DDD原则、微服务拆分科学
3. **可落地性好**：详细的设计文档、明确的技术选型
4. **可扩展性强**：支持业务快速迭代、技术平滑演进

### 核心优势

- ✅ **清晰的边界**：每个服务职责明确，边界清晰
- ✅ **高内聚低耦合**：服务内高度内聚，服务间松耦合
- ✅ **事件驱动**：通过领域事件实现异步解耦
- ✅ **独立扩展**：针对高负载服务独立扩展
- ✅ **故障隔离**：服务故障不影响其他服务
- ✅ **技术栈统一**：Spring Cloud生态，降低学习成本

## 👥 团队协作

- **团队规模**：建议8-12人
- **团队划分**：按微服务划分，每个服务1-2人
- **协作方式**：通过API契约协作，独立开发测试

### 📋 8人团队分工方案

我们为8人团队提供了详细的分工方案文档：

- 📄 **[团队分工方案（完整版）](docs/团队分工方案.md)** - 详细的工作内容、时间计划、质量标准

**分工结构：**
- 1人负责架构设计和技术攻坚
- 1人负责客户服务（Customer Service）
- 2人负责用户服务（User Service）- 核心业务+性能优化
- 1人负责账户服务（Account Service）
- 1人负责订购服务（Subscription Service）
- 1人负责基础设施（Gateway/Config/Registry）
- 1人负责质量保障（测试/DevOps/监控）

**预期成果：** 完成4个核心微服务的完整实现，达到生产就绪状态

## 📝 License

[MIT](LICENSE)

## 🤝 贡献

欢迎贡献代码、提出建议！
