# 电信CRM客户中心 - REST API 接口设计

## 一、概述

本文档详细定义客户中心各微服务的REST API接口设计，包括请求格式、响应格式、错误处理等。

## 二、接口设计规范

### 2.1 通用规范

**URL规范：**
- 使用RESTful风格
- 使用小写字母和连字符（kebab-case）
- 资源名称使用复数形式
- 版本号放在URL中：`/api/v1/...`

**HTTP方法：**
- GET：查询资源
- POST：创建资源
- PUT：完整更新资源
- PATCH：部分更新资源
- DELETE：删除资源

**状态码：**
- 200 OK：成功
- 201 Created：创建成功
- 204 No Content：删除成功
- 400 Bad Request：请求参数错误
- 401 Unauthorized：未认证
- 403 Forbidden：无权限
- 404 Not Found：资源不存在
- 409 Conflict：资源冲突
- 500 Internal Server Error：服务器错误

### 2.2 请求格式

**请求头（Headers）：**
```
Content-Type: application/json
Authorization: Bearer {JWT_TOKEN}
X-Request-ID: {UUID}  # 请求追踪ID
X-Channel: {CHANNEL}   # 渠道：WEB/APP/USSD/CALL_CENTER
```

**请求体（Body）：**
```json
{
  "data": {
    // 业务数据
  }
}
```

### 2.3 响应格式

**成功响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    // 业务数据
  },
  "requestId": "550e8400-e29b-41d4-a716-446655440000",
  "timestamp": "2024-01-01T12:00:00Z"
}
```

**分页响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "items": [],
    "page": 1,
    "pageSize": 20,
    "total": 100,
    "totalPages": 5
  },
  "requestId": "...",
  "timestamp": "..."
}
```

**错误响应：**
```json
{
  "code": 40001,
  "message": "参数校验失败",
  "errors": [
    {
      "field": "phoneNumber",
      "message": "手机号码格式不正确"
    }
  ],
  "requestId": "...",
  "timestamp": "..."
}
```

### 2.4 错误码设计

| 错误码 | 说明 |
|-------|------|
| 0 | 成功 |
| 10001-19999 | 客户服务错误 |
| 20001-29999 | 用户服务错误 |
| 30001-39999 | 账户服务错误 |
| 40001-49999 | 订购服务错误 |
| 90001-99999 | 系统级错误 |

## 三、客户管理服务接口

### 3.1 创建个人客户

**接口描述：** 创建个人客户

**请求：**
```
POST /api/v1/customers/individual
Content-Type: application/json
Authorization: Bearer {JWT_TOKEN}
```

**请求体：**
```json
{
  "name": "张三",
  "idType": "ID_CARD",
  "idNumber": "110101199001011234",
  "gender": "MALE",
  "birthDate": "1990-01-01",
  "contactPhone": "13800138000",
  "email": "zhangsan@example.com",
  "address": {
    "province": "北京市",
    "city": "北京市",
    "district": "朝阳区",
    "street": "朝阳街道",
    "detailAddress": "朝阳路1号",
    "postalCode": "100000"
  }
}
```

**响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "customerId": 1001,
    "customerType": "INDIVIDUAL",
    "status": "ACTIVE",
    "level": 1,
    "points": 0,
    "createdTime": "2024-01-01T12:00:00Z"
  },
  "requestId": "550e8400-e29b-41d4-a716-446655440000",
  "timestamp": "2024-01-01T12:00:00Z"
}
```

**错误码：**
- 10001: 证件号码已存在
- 10002: 实名认证失败
- 10003: 超出一证五户限制

---

### 3.2 查询客户信息

**接口描述：** 根据客户ID查询客户详细信息

**请求：**
```
GET /api/v1/customers/{customerId}
Authorization: Bearer {JWT_TOKEN}
```

**路径参数：**
- `customerId`: 客户ID

**响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "customerId": 1001,
    "customerType": "INDIVIDUAL",
    "status": "ACTIVE",
    "level": 1,
    "points": 100,
    "profile": {
      "name": "张三",
      "idType": "ID_CARD",
      "idNumber": "110101199001011234",
      "gender": "MALE",
      "birthDate": "1990-01-01",
      "contactPhone": "13800138000",
      "email": "zhangsan@example.com",
      "address": {
        "province": "北京市",
        "city": "北京市",
        "district": "朝阳区",
        "street": "朝阳街道",
        "detailAddress": "朝阳路1号",
        "postalCode": "100000"
      }
    },
    "createdTime": "2024-01-01T12:00:00Z",
    "updatedTime": "2024-01-01T12:00:00Z"
  }
}
```

**错误码：**
- 10404: 客户不存在

---

### 3.3 更新客户资料

**接口描述：** 更新客户基本资料

**请求：**
```
PUT /api/v1/customers/{customerId}/profile
Content-Type: application/json
Authorization: Bearer {JWT_TOKEN}
```

**请求体：**
```json
{
  "contactPhone": "13900139000",
  "email": "zhangsan_new@example.com",
  "address": {
    "province": "北京市",
    "city": "北京市",
    "district": "海淀区",
    "street": "中关村街道",
    "detailAddress": "中关村大街1号",
    "postalCode": "100080"
  }
}
```

**响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "customerId": 1001,
    "updatedTime": "2024-01-02T12:00:00Z"
  }
}
```

---

## 四、用户管理服务接口

### 4.1 创建用户

**接口描述：** 创建个人用户（开户）

**请求：**
```
POST /api/v1/users
Content-Type: application/json
Authorization: Bearer {JWT_TOKEN}
```

**请求体：**
```json
{
  "customerId": 1001,
  "phoneNumber": "13800138000",
  "userType": "INDIVIDUAL",
  "packageId": "PKG-001",
  "simCard": {
    "iccid": "89860000000000000001",
    "imsi": "460000000000001",
    "cardType": "5G"
  }
}
```

**响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "userId": 10001,
    "phoneNumber": "13800138000",
    "customerId": 1001,
    "userType": "INDIVIDUAL",
    "status": "PRE_ACTIVE",
    "simCard": {
      "simCardId": 20001,
      "iccid": "89860000000000000001",
      "imsi": "460000000000001",
      "cardType": "5G",
      "status": "NORMAL"
    },
    "servicePackage": {
      "packageId": "PKG-001",
      "packageName": "5G畅享套餐",
      "monthlyFee": 99.00,
      "includedTrafficMb": 30720,
      "includedVoiceMin": 1000,
      "includedSms": 100
    },
    "openTime": "2024-01-01T12:00:00Z"
  }
}
```

**错误码：**
- 20001: 客户不存在
- 20002: 手机号码已被使用
- 20003: 套餐不存在
- 20004: 网络开通失败

---

### 4.2 用户激活

**接口描述：** 激活用户（首次使用）

**请求：**
```
POST /api/v1/users/{userId}/activate
Authorization: Bearer {JWT_TOKEN}
```

**请求体：**
```json
{}
```

**响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "userId": 10001,
    "status": "ACTIVE",
    "activeTime": "2024-01-01T13:00:00Z"
  }
}
```

**错误码：**
- 20101: 用户不存在
- 20102: 用户状态不允许激活

---

### 4.3 变更套餐

**接口描述：** 变更用户套餐

**请求：**
```
POST /api/v1/users/{userId}/change-package
Content-Type: application/json
Authorization: Bearer {JWT_TOKEN}
```

**请求体：**
```json
{
  "newPackageId": "PKG-002",
  "effectiveType": "NEXT_MONTH"
}
```

**请求参数说明：**
- `newPackageId`: 新套餐ID
- `effectiveType`: 生效类型
  - `IMMEDIATE`: 立即生效
  - `NEXT_MONTH`: 次月生效

**响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "userId": 10001,
    "oldPackage": {
      "packageId": "PKG-001",
      "packageName": "5G畅享套餐"
    },
    "newPackage": {
      "packageId": "PKG-002",
      "packageName": "5G尊享套餐"
    },
    "effectiveType": "NEXT_MONTH",
    "effectiveTime": "2024-02-01T00:00:00Z",
    "fee": 0.00
  }
}
```

**错误码：**
- 20201: 用户不存在
- 20202: 套餐不存在
- 20203: 套餐互斥，需先退订冲突产品
- 20204: 缺少依赖产品
- 20205: 账户余额不足（立即生效需要补差费）

---

### 4.4 用户停机

**接口描述：** 停机用户

**请求：**
```
POST /api/v1/users/{userId}/suspend
Content-Type: application/json
Authorization: Bearer {JWT_TOKEN}
```

**请求体：**
```json
{
  "reason": "USER_REQUEST",
  "remark": "用户申请停机"
}
```

**响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "userId": 10001,
    "status": "SUSPENDED_REPORT",
    "suspendTime": "2024-01-01T14:00:00Z"
  }
}
```

---

### 4.5 用户复机

**接口描述：** 复机用户

**请求：**
```
POST /api/v1/users/{userId}/resume
Authorization: Bearer {JWT_TOKEN}
```

**响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "userId": 10001,
    "status": "ACTIVE",
    "resumeTime": "2024-01-02T10:00:00Z"
  }
}
```

**错误码：**
- 20301: 用户不存在
- 20302: 用户状态不允许复机
- 20303: 欠费未结清，不能复机

---

### 4.6 查询用户信息

**接口描述：** 根据用户ID或手机号码查询用户信息

**请求1（按用户ID查询）：**
```
GET /api/v1/users/{userId}
Authorization: Bearer {JWT_TOKEN}
```

**请求2（按手机号码查询）：**
```
GET /api/v1/users?phoneNumber=13800138000
Authorization: Bearer {JWT_TOKEN}
```

**响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "userId": 10001,
    "phoneNumber": "13800138000",
    "customerId": 1001,
    "userType": "INDIVIDUAL",
    "status": "ACTIVE",
    "simCard": {
      "simCardId": 20001,
      "iccid": "89860000000000000001",
      "imsi": "460000000000001",
      "cardType": "5G",
      "status": "NORMAL"
    },
    "servicePackage": {
      "packageId": "PKG-001",
      "packageName": "5G畅享套餐",
      "monthlyFee": 99.00,
      "includedTrafficMb": 30720,
      "includedVoiceMin": 1000,
      "includedSms": 100,
      "effectiveTime": "2024-01-01T00:00:00Z"
    },
    "openTime": "2024-01-01T12:00:00Z",
    "activeTime": "2024-01-01T13:00:00Z"
  }
}
```

---

## 五、账户管理服务接口

### 5.1 创建账户

**接口描述：** 创建账户

**请求：**
```
POST /api/v1/accounts
Content-Type: application/json
Authorization: Bearer {JWT_TOKEN}
```

**请求体：**
```json
{
  "customerId": 1001,
  "accountType": "PREPAID"
}
```

**响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "accountId": 30001,
    "customerId": 1001,
    "accountType": "PREPAID",
    "balance": 0.00,
    "frozenBalance": 0.00,
    "creditLimit": 0.00,
    "status": "ACTIVE",
    "openTime": "2024-01-01T12:00:00Z"
  }
}
```

---

### 5.2 账户充值

**接口描述：** 账户充值

**请求：**
```
POST /api/v1/accounts/{accountId}/recharge
Content-Type: application/json
Authorization: Bearer {JWT_TOKEN}
```

**请求体：**
```json
{
  "amount": 100.00,
  "paymentMethod": "ALIPAY",
  "channel": "APP"
}
```

**响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "transactionId": "TXN20240101120000001",
    "accountId": 30001,
    "amount": 100.00,
    "balanceBefore": 0.00,
    "balanceAfter": 100.00,
    "transactionTime": "2024-01-01T12:00:00Z"
  }
}
```

---

### 5.3 账户扣费

**接口描述：** 账户扣费（供计费中心调用）

**请求：**
```
POST /api/v1/accounts/{accountId}/deduct
Content-Type: application/json
Authorization: Bearer {JWT_TOKEN}
```

**请求体：**
```json
{
  "amount": 99.00,
  "reason": "月租费",
  "relatedOrderId": "ORDER20240101001"
}
```

**响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "transactionId": "TXN20240101120000002",
    "accountId": 30001,
    "amount": 99.00,
    "balanceBefore": 100.00,
    "balanceAfter": 1.00,
    "transactionTime": "2024-01-01T12:00:00Z"
  }
}
```

**错误码：**
- 30201: 账户不存在
- 30202: 账户余额不足

---

### 5.4 绑定用户到账户

**接口描述：** 将用户绑定到账户，用于费用扣除

**请求：**
```
POST /api/v1/accounts/{accountId}/bind-user
Content-Type: application/json
Authorization: Bearer {JWT_TOKEN}
```

**请求体：**
```json
{
  "userId": 10001,
  "relationshipType": "PRIMARY",
  "priority": 1
}
```

**响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "relationshipId": 40001,
    "accountId": 30001,
    "userId": 10001,
    "relationshipType": "PRIMARY",
    "priority": 1,
    "effectiveTime": "2024-01-01T12:00:00Z"
  }
}
```

---

### 5.5 查询账户余额

**接口描述：** 查询账户余额

**请求：**
```
GET /api/v1/accounts/{accountId}/balance
Authorization: Bearer {JWT_TOKEN}
```

**响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "accountId": 30001,
    "balance": 1.00,
    "frozenBalance": 0.00,
    "availableBalance": 1.00,
    "creditLimit": 0.00
  }
}
```

---

### 5.6 查询交易记录

**接口描述：** 查询账户交易记录

**请求：**
```
GET /api/v1/accounts/{accountId}/transactions?page=1&pageSize=20
Authorization: Bearer {JWT_TOKEN}
```

**响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "items": [
      {
        "transactionId": "TXN20240101120000002",
        "accountId": 30001,
        "transactionType": "DEDUCTION",
        "amount": 99.00,
        "balanceBefore": 100.00,
        "balanceAfter": 1.00,
        "transactionTime": "2024-01-01T12:00:00Z",
        "description": "月租费",
        "relatedOrderId": "ORDER20240101001"
      },
      {
        "transactionId": "TXN20240101120000001",
        "accountId": 30001,
        "transactionType": "RECHARGE",
        "amount": 100.00,
        "balanceBefore": 0.00,
        "balanceAfter": 100.00,
        "transactionTime": "2024-01-01T12:00:00Z",
        "channel": "APP",
        "description": "账户充值"
      }
    ],
    "page": 1,
    "pageSize": 20,
    "total": 2,
    "totalPages": 1
  }
}
```

---

## 六、产品订购服务接口

### 6.1 订购产品

**接口描述：** 订购产品（增值业务、流量包等）

**请求：**
```
POST /api/v1/subscriptions
Content-Type: application/json
Authorization: Bearer {JWT_TOKEN}
```

**请求体：**
```json
{
  "userId": 10001,
  "productId": "PROD-001",
  "effectiveType": "IMMEDIATE"
}
```

**响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "subscriptionId": 50001,
    "userId": 10001,
    "productId": "PROD-001",
    "productName": "10GB流量包",
    "subscriptionType": "VALUE_ADDED",
    "status": "ACTIVE",
    "subscribeTime": "2024-01-01T12:00:00Z",
    "effectiveTime": "2024-01-01T12:00:00Z",
    "expiryTime": "2024-02-01T00:00:00Z"
  }
}
```

**错误码：**
- 40001: 用户不存在
- 40002: 产品不存在
- 40003: 产品互斥，需先退订冲突产品
- 40004: 缺少依赖产品

---

### 6.2 退订产品

**接口描述：** 退订产品

**请求：**
```
DELETE /api/v1/subscriptions/{subscriptionId}
Authorization: Bearer {JWT_TOKEN}
```

**响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "subscriptionId": 50001,
    "status": "CANCELLED",
    "cancelTime": "2024-01-15T12:00:00Z"
  }
}
```

---

### 6.3 查询用户订购列表

**接口描述：** 查询用户已订购的产品列表

**请求：**
```
GET /api/v1/subscriptions?userId=10001&status=ACTIVE
Authorization: Bearer {JWT_TOKEN}
```

**响应：**
```json
{
  "code": 0,
  "message": "success",
  "data": {
    "items": [
      {
        "subscriptionId": 50001,
        "userId": 10001,
        "productId": "PROD-001",
        "productName": "10GB流量包",
        "subscriptionType": "VALUE_ADDED",
        "status": "ACTIVE",
        "subscribeTime": "2024-01-01T12:00:00Z",
        "effectiveTime": "2024-01-01T12:00:00Z",
        "expiryTime": "2024-02-01T00:00:00Z"
      }
    ],
    "page": 1,
    "pageSize": 20,
    "total": 1,
    "totalPages": 1
  }
}
```

---

## 七、跨中心集成接口

### 7.1 与产商品中心集成

**查询产品信息：**
```
GET /api/v1/products/{productId}
Authorization: Bearer {JWT_TOKEN}
```

**查询套餐列表：**
```
GET /api/v1/products?type=PACKAGE&status=AVAILABLE
Authorization: Bearer {JWT_TOKEN}
```

---

### 7.2 与开通中心集成

**用户开通：**
```
POST /api/v1/provisioning/users
Content-Type: application/json
Authorization: Bearer {JWT_TOKEN}

{
  "userId": 10001,
  "phoneNumber": "13800138000",
  "imsi": "460000000000001",
  "packageId": "PKG-001"
}
```

**用户停机：**
```
POST /api/v1/provisioning/users/{userId}/suspend
Authorization: Bearer {JWT_TOKEN}
```

**用户复机：**
```
POST /api/v1/provisioning/users/{userId}/resume
Authorization: Bearer {JWT_TOKEN}
```

---

### 7.3 与计费中心集成

**查询用户账单：**
```
GET /api/v1/billing/users/{userId}/bills?month=202401
Authorization: Bearer {JWT_TOKEN}
```

**批量扣费：**
```
POST /api/v1/billing/batch-deduct
Content-Type: application/json
Authorization: Bearer {JWT_TOKEN}

{
  "deductions": [
    {
      "userId": 10001,
      "accountId": 30001,
      "amount": 99.00,
      "reason": "月租费"
    }
  ]
}
```

---

## 八、接口安全

### 8.1 认证方式

使用JWT (JSON Web Token) 进行认证：

1. 客户端登录后获取JWT Token
2. 每次请求在Header中携带Token
3. 服务端验证Token的有效性

**JWT Token格式：**
```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**JWT Payload：**
```json
{
  "sub": "user123",
  "role": "CUSTOMER",
  "exp": 1704096000,
  "iat": 1704088800
}
```

### 8.2 接口权限

不同角色有不同的接口权限：

| 角色 | 权限 |
|------|------|
| CUSTOMER | 只能操作自己的数据 |
| EMPLOYEE | 可以操作所有客户的数据 |
| ADMIN | 系统管理权限 |

### 8.3 数据脱敏

敏感数据在返回时需要脱敏：

- **证件号码**：`110101********1234`
- **手机号码**：`138****8000`
- **姓名**：`张*`

### 8.4 限流

使用令牌桶算法进行限流：

- 单IP限流：100次/分钟
- 单用户限流：1000次/分钟
- 单接口限流：根据接口特性设置

---

## 九、接口测试

### 9.1 Postman集合

提供Postman集合文件，包含所有接口的测试用例。

### 9.2 接口文档

使用Swagger/OpenAPI生成在线接口文档：

```
http://api-gateway:8080/swagger-ui.html
```

---

## 十、总结

本接口设计文档定义了客户中心各微服务的REST API接口，具有以下特点：

1. **RESTful规范**：遵循REST设计原则
2. **统一格式**：请求和响应格式统一
3. **完善的错误处理**：明确的错误码和错误信息
4. **安全性**：JWT认证、权限控制、数据脱敏
5. **可扩展性**：版本号管理，支持接口演进

这些接口可以支撑前端各种渠道（Web、APP、USSD、Call Center）的接入。
