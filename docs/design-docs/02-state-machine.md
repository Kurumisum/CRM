# 电信CRM客户中心 - 核心实体状态机

## 一、概述

状态机（State Machine）是描述实体生命周期和状态转换的重要工具。本文档定义了客户中心核心实体的状态机，包括状态定义、转换条件和转换动作。

## 二、客户状态机

### 2.1 客户状态定义

| 状态码 | 状态名称 | 英文 | 描述 |
|--------|---------|------|------|
| 00 | 正常 | ACTIVE | 客户状态正常，可以办理各类业务 |
| 01 | 欠费 | ARREARS | 客户有欠费，提醒缴费 |
| 02 | 停机 | SUSPENDED | 客户因长期欠费被停机 |
| 99 | 销户 | CLOSED | 客户已销户，终止服务 |

### 2.2 客户状态转换图

```
                    ┌─────────────┐
                    │   [初始]    │
                    └──────┬──────┘
                           │ 客户开户
                           │
                           ▼
┌──────────────────► ┌─────────────┐
│                    │   正常      │ ◄────────────┐
│    缴清欠费         │  (ACTIVE)   │              │
│                    └──────┬──────┘              │
│                           │                     │
│                           │ 产生欠费             │
│                           │                     │
│                           ▼                     │
│                    ┌─────────────┐              │
│                    │   欠费      │              │
└────────────────────┤  (ARREARS)  │              │
     缴清欠费        └──────┬──────┘              │
                            │                     │
                            │ 长期欠费             │
                            │                     │
                            ▼                     │
                     ┌─────────────┐              │
                     │   停机      │              │
                     │ (SUSPENDED) │──────────────┘
                     └──────┬──────┘    缴清欠费
                            │
                            │ 申请销户
                            │
                            ▼
                     ┌─────────────┐
                     │   销户      │
                     │  (CLOSED)   │
                     └─────────────┘
                            │
                            │ [终态]
                            ▼
```

### 2.3 客户状态转换表

| 当前状态 | 事件 | 前置条件 | 后置条件 | 目标状态 | 转换动作 |
|---------|------|---------|---------|---------|---------|
| [初始] | 客户开户 | 通过实名认证 | 客户档案创建 | 正常 | - 生成客户ID<br>- 创建客户档案<br>- 发送CustomerCreatedEvent |
| 正常 | 产生欠费 | 账户余额不足 | 欠费记录生成 | 欠费 | - 记录欠费金额<br>- 发送欠费通知<br>- 发送CustomerStatusChangedEvent |
| 欠费 | 缴清欠费 | 欠费金额结清 | 欠费记录清除 | 正常 | - 清除欠费标记<br>- 发送CustomerStatusChangedEvent |
| 欠费 | 长期欠费 | 欠费超过N天 | 服务停止 | 停机 | - 停止所有用户服务<br>- 发送停机通知<br>- 发送CustomerStatusChangedEvent |
| 停机 | 缴清欠费 | 欠费金额结清 | 服务恢复 | 正常 | - 恢复用户服务<br>- 发送CustomerStatusChangedEvent |
| 停机 | 申请销户 | 欠费结清 | 客户注销 | 销户 | - 注销所有用户<br>- 关闭所有账户<br>- 发送CustomerClosedEvent |
| 正常 | 申请销户 | 无欠费且所有用户已注销 | 客户注销 | 销户 | - 结算余额退费<br>- 关闭所有账户<br>- 发送CustomerClosedEvent |

### 2.4 客户状态转换约束

1. **不可逆转换**：销户状态是终态，不可恢复
2. **欠费处理**：欠费超过90天自动转为停机
3. **销户前提**：必须先注销所有用户
4. **余额处理**：销户时退还所有账户余额

## 三、用户状态机

### 3.1 用户状态定义

| 状态码 | 状态名称 | 英文 | 描述 |
|--------|---------|------|------|
| 0 | 预开 | PRE_ACTIVE | 已开户但未激活，不产生费用 |
| 1 | 正常 | ACTIVE | 正常使用中 |
| 2 | 欠费停机 | SUSPENDED_ARREARS | 因账户欠费被系统停机 |
| 3 | 申请停机 | SUSPENDED_REPORT | 用户主动申请停机 |
| 8 | 预销户 | PRE_TERMINATION | 已申请销户，等待期中（30天） |
| 9 | 已销户 | TERMINATED | 已注销，号码进入回收池 |

### 3.2 用户状态转换图

```
                    ┌─────────────┐
                    │   [初始]    │
                    └──────┬──────┘
                           │ 用户开户
                           │
                           ▼
                    ┌─────────────┐
                    │   预开      │
                    │(PRE_ACTIVE) │
                    └──────┬──────┘
                           │ 首次激活
                           │
                           ▼
┌──────────────────► ┌─────────────┐ ◄────────────┐
│                    │   正常      │              │
│    复机             │  (ACTIVE)   │   复机       │
│                    └──┬───────┬──┘              │
│                       │       │                 │
│        账户欠费       │       │ 用户申请停机     │
│                       │       │                 │
│                       ▼       ▼                 │
│                ┌─────────┐ ┌─────────┐          │
│                │ 欠费停机 │ │申请停机 │          │
│                │(SUSP_AR)│ │(SUSP_RE)│          │
│                └────┬────┘ └────┬────┘          │
│                     │           │               │
│                     │           │               │
│                     └─────┬─────┘               │
│                           │                     │
│                           │ 申请销户             │
│                           │                     │
│                           ▼                     │
│                    ┌─────────────┐              │
│                    │   预销户    │──────────────┘
│                    │(PRE_TERMIN) │  30天内取消
│                    └──────┬──────┘
│                           │
│                           │ 30天后自动确认
│                           │
│                           ▼
│                    ┌─────────────┐
│                    │  已销户     │
│                    │(TERMINATED) │
│                    └─────────────┘
│                           │
└───────────────────────────┘ [终态]
        号码回收（6个月后）
```

### 3.3 用户状态转换表

| 当前状态 | 事件 | 前置条件 | 后置条件 | 目标状态 | 转换动作 |
|---------|------|---------|---------|---------|---------|
| [初始] | 用户开户 | - 客户状态正常<br>- 号码可用<br>- 网络资源开通成功 | 用户记录创建 | 预开 | - 生成用户ID<br>- 分配号码<br>- 关联SIM卡<br>- 发送UserOpenedEvent |
| 预开 | 首次激活 | SIM卡插入手机并开机 | 用户激活 | 正常 | - 记录激活时间<br>- 开始计费<br>- 发送UserActivatedEvent |
| 正常 | 账户欠费 | - 绑定的账户余额不足<br>- 欠费超过7天 | 服务停止 | 欠费停机 | - 停止语音、流量、短信服务<br>- 发送欠费通知<br>- 发送UserStatusChangedEvent |
| 正常 | 用户申请停机 | 用户主动申请 | 服务暂停 | 申请停机 | - 停止服务<br>- 暂停计费<br>- 发送UserStatusChangedEvent |
| 欠费停机 | 复机 | 欠费已结清 | 服务恢复 | 正常 | - 恢复服务<br>- 恢复计费<br>- 发送UserStatusChangedEvent |
| 申请停机 | 复机 | 用户申请复机 | 服务恢复 | 正常 | - 恢复服务<br>- 恢复计费<br>- 发送UserStatusChangedEvent |
| 正常 | 申请销户 | 无欠费 | 进入等待期 | 预销户 | - 设置销户日期（30天后）<br>- 发送UserStatusChangedEvent |
| 欠费停机 | 申请销户 | 欠费已结清 | 进入等待期 | 预销户 | - 设置销户日期（30天后）<br>- 发送UserStatusChangedEvent |
| 申请停机 | 申请销户 | 无欠费 | 进入等待期 | 预销户 | - 设置销户日期（30天后）<br>- 发送UserStatusChangedEvent |
| 预销户 | 取消销户 | 在30天等待期内 | 取消销户 | 正常 | - 清除销户日期<br>- 发送UserStatusChangedEvent |
| 预销户 | 确认销户 | 30天等待期已过 | 用户注销 | 已销户 | - 注销用户<br>- 回收号码<br>- 发送UserClosedEvent |

### 3.4 用户状态转换约束

1. **激活时机**：预开状态下首次使用服务时自动激活
2. **停机保护**：停机期间不计算月租费，但保留号码
3. **复机条件**：欠费停机必须先结清欠费才能复机
4. **销户等待期**：预销户期间（30天）用户可以反悔
5. **号码回收**：销户后号码冷冻6个月后重新投入号码池
6. **不可逆转换**：已销户状态是终态，不可恢复

## 四、SIM卡状态机

### 4.1 SIM卡状态定义

| 状态码 | 状态名称 | 英文 | 描述 |
|--------|---------|------|------|
| 1 | 正常 | NORMAL | 正常使用中 |
| 2 | 挂失 | LOST | 已挂失，临时停止服务 |
| 3 | 损坏 | DAMAGED | 卡损坏，需要换卡 |
| 9 | 失效 | INVALID | 已失效，不可使用 |

### 4.2 SIM卡状态转换图

```
                    ┌─────────────┐
                    │   [初始]    │
                    └──────┬──────┘
                           │ 发卡
                           │
                           ▼
┌──────────────────► ┌─────────────┐
│                    │   正常      │
│      解挂           │  (NORMAL)   │
│                    └──┬───────┬──┘
│                       │       │
│          挂失         │       │ 损坏
│                       │       │
│                       ▼       ▼
│                ┌─────────┐ ┌─────────┐
│                │  挂失   │ │  损坏   │
│                │ (LOST)  │ │(DAMAGED)│
│                └────┬────┘ └────┬────┘
│                     │           │
└─────────────────────┘           │
                                  │ 换卡
                                  │
                                  ▼
                           ┌─────────────┐
                           │   失效      │
                           │  (INVALID)  │
                           └─────────────┘
                                  │
                                  │ [终态]
                                  ▼
```

### 4.3 SIM卡状态转换表

| 当前状态 | 事件 | 前置条件 | 后置条件 | 目标状态 | 转换动作 |
|---------|------|---------|---------|---------|---------|
| [初始] | 发卡 | 用户开户 | SIM卡激活 | 正常 | - 写入IMSI到HLR<br>- 关联用户<br>- 发送SimCardIssuedEvent |
| 正常 | 挂失 | 用户申请 | 服务临时停止 | 挂失 | - 停止服务<br>- 标记挂失<br>- 发送SimCardLostEvent |
| 挂失 | 解挂 | 用户申请 | 服务恢复 | 正常 | - 恢复服务<br>- 清除挂失标记<br>- 发送SimCardUnlostEvent |
| 正常 | 损坏 | 卡物理损坏 | 需要换卡 | 损坏 | - 标记损坏<br>- 停止服务 |
| 挂失 | 换卡 | 补卡或换卡 | 新卡激活 | 失效 | - 作废旧卡<br>- 激活新卡<br>- 发送SimCardReplacedEvent |
| 损坏 | 换卡 | 换卡 | 新卡激活 | 失效 | - 作废旧卡<br>- 激活新卡<br>- 发送SimCardReplacedEvent |

## 五、账户状态机

### 5.1 账户状态定义

| 状态码 | 状态名称 | 英文 | 描述 |
|--------|---------|------|------|
| 1 | 正常 | ACTIVE | 账户正常，可以充值和扣费 |
| 2 | 冻结 | FROZEN | 账户冻结，不能操作 |
| 9 | 已关闭 | CLOSED | 账户关闭 |

### 5.2 账户状态转换图

```
                    ┌─────────────┐
                    │   [初始]    │
                    └──────┬──────┘
                           │ 账户开立
                           │
                           ▼
┌──────────────────► ┌─────────────┐
│                    │   正常      │
│      解冻           │  (ACTIVE)   │
│                    └──────┬──────┘
│                           │
│                           │ 账户冻结
│                           │ (如：司法冻结)
│                           │
│                           ▼
│                    ┌─────────────┐
│                    │   冻结      │
└────────────────────┤  (FROZEN)   │
                     └──────┬──────┘
                            │
                            │ 账户关闭
                            │
                            ▼
                     ┌─────────────┐
                     │  已关闭     │
                     │  (CLOSED)   │
                     └─────────────┘
                            │
                            │ [终态]
                            ▼
```

### 5.3 账户状态转换表

| 当前状态 | 事件 | 前置条件 | 后置条件 | 目标状态 | 转换动作 |
|---------|------|---------|---------|---------|---------|
| [初始] | 账户开立 | 客户状态正常 | 账户创建 | 正常 | - 生成账户ID<br>- 初始化余额为0<br>- 发送AccountOpenedEvent |
| 正常 | 账户冻结 | 司法冻结/风控冻结 | 账户锁定 | 冻结 | - 标记冻结<br>- 禁止操作<br>- 发送AccountFrozenEvent |
| 冻结 | 解冻 | 冻结原因解除 | 账户解锁 | 正常 | - 清除冻结标记<br>- 恢复操作<br>- 发送AccountUnfrozenEvent |
| 正常 | 账户关闭 | - 无绑定用户<br>- 余额已退还 | 账户注销 | 已关闭 | - 注销账户<br>- 发送AccountClosedEvent |
| 冻结 | 账户关闭 | - 无绑定用户<br>- 冻结原因解除<br>- 余额已退还 | 账户注销 | 已关闭 | - 解冻<br>- 注销账户<br>- 发送AccountClosedEvent |

### 5.4 账户状态转换约束

1. **冻结类型**：司法冻结、风控冻结、用户自主冻结
2. **冻结限制**：冻结期间不能充值、扣费、提现
3. **关闭前提**：必须先解绑所有用户
4. **余额处理**：关闭前必须退还所有余额
5. **不可逆转换**：已关闭状态是终态，不可恢复

## 六、产品订购状态机

### 6.1 产品订购状态定义

| 状态码 | 状态名称 | 英文 | 描述 |
|--------|---------|------|------|
| 0 | 待生效 | PENDING | 已订购但未生效 |
| 1 | 生效中 | ACTIVE | 正常使用中 |
| 2 | 已暂停 | SUSPENDED | 暂停使用 |
| 8 | 已失效 | EXPIRED | 订购期已过 |
| 9 | 已取消 | CANCELLED | 用户退订 |

### 6.2 产品订购状态转换图

```
                    ┌─────────────┐
                    │   [初始]    │
                    └──────┬──────┘
                           │ 订购产品
                           │
                           ▼
                    ┌─────────────┐
                    │   待生效    │
                    │  (PENDING)  │
                    └──────┬──────┘
                           │ 生效时间到达
                           │
                           ▼
┌──────────────────► ┌─────────────┐
│                    │   生效中    │
│      恢复           │  (ACTIVE)   │
│                    └──┬───────┬──┘
│                       │       │
│          暂停         │       │ 订购期结束
│                       │       │
│                       ▼       ▼
│                ┌─────────┐ ┌─────────┐
│                │  已暂停 │ │ 已失效  │
│                │(SUSPEND)│ │(EXPIRED)│
│                └────┬────┘ └─────────┘
│                     │           │
└─────────────────────┘           │
                                  │ [终态]
         用户退订                  ▼
              │
              ▼
       ┌─────────────┐
       │  已取消     │
       │(CANCELLED)  │
       └─────────────┘
              │
              │ [终态]
              ▼
```

### 6.3 产品订购状态转换表

| 当前状态 | 事件 | 前置条件 | 后置条件 | 目标状态 | 转换动作 |
|---------|------|---------|---------|---------|---------|
| [初始] | 订购产品 | - 用户状态正常<br>- 通过订购规则检查 | 订购记录创建 | 待生效 | - 创建订购记录<br>- 设置生效时间<br>- 发送ProductSubscribedEvent |
| 待生效 | 生效时间到达 | 系统定时检查 | 产品激活 | 生效中 | - 激活产品<br>- 通知开通中心<br>- 发送SubscriptionActivatedEvent |
| 生效中 | 暂停 | 用户申请或欠费 | 产品暂停 | 已暂停 | - 暂停产品<br>- 发送SubscriptionSuspendedEvent |
| 已暂停 | 恢复 | 用户申请或缴费 | 产品恢复 | 生效中 | - 恢复产品<br>- 发送SubscriptionResumedEvent |
| 生效中 | 订购期结束 | 到期且不续期 | 产品失效 | 已失效 | - 失效产品<br>- 发送SubscriptionExpiredEvent |
| 待生效 | 用户退订 | 用户申请 | 订购取消 | 已取消 | - 取消订购<br>- 退还费用<br>- 发送ProductUnsubscribedEvent |
| 生效中 | 用户退订 | 用户申请 | 订购取消 | 已取消 | - 取消订购<br>- 计算退费<br>- 发送ProductUnsubscribedEvent |

### 6.4 产品订购状态转换约束

1. **生效规则**：立即生效或次月1日生效
2. **暂停原因**：用户申请、账户欠费、违规使用
3. **失效处理**：失效后自动关闭，需要重新订购
4. **退订规则**：部分产品有最短订购期限制
5. **退费计算**：按照产品退费规则计算

## 七、状态机设计原则

### 7.1 状态设计原则
1. **状态互斥**：实体在任何时刻只能处于一种状态
2. **状态完备**：覆盖实体生命周期的所有阶段
3. **状态清晰**：每个状态有明确的业务含义
4. **终态唯一**：终态不可再转换到其他状态

### 7.2 转换设计原则
1. **转换合法**：只能从定义的状态转换到目标状态
2. **条件明确**：每个转换有清晰的前置条件
3. **动作原子**：转换动作应该是原子性的
4. **事件发布**：状态转换后发布领域事件

### 7.3 实现建议
1. **状态枚举**：使用枚举类型定义状态
2. **状态模式**：使用状态模式实现复杂的状态转换逻辑
3. **状态持久化**：状态变更需要持久化到数据库
4. **并发控制**：使用乐观锁防止并发状态变更冲突
5. **状态审计**：记录所有状态变更历史

### 7.4 异常处理
1. **非法转换**：尝试非法状态转换时抛出异常
2. **前置条件不满足**：检查前置条件，不满足时拒绝转换
3. **转换失败**：转换过程中失败应该回滚，保持原状态
4. **补偿机制**：关键转换失败时有补偿措施

## 八、状态机实现示例（伪代码）

### 8.1 用户状态机实现

```java
public class User extends AggregateRoot {
    private UserId userId;
    private UserStatus status;
    private Long version; // 乐观锁版本号
    
    // 激活用户
    public void activate() {
        if (this.status != UserStatus.PRE_ACTIVE) {
            throw new IllegalStateException(
                "只有预开状态的用户才能激活");
        }
        
        this.status = UserStatus.ACTIVE;
        this.activeTime = DateTime.now();
        
        // 发布领域事件
        this.addDomainEvent(new UserActivatedEvent(
            this.userId, this.activeTime));
    }
    
    // 停机
    public void suspend(SuspendReason reason) {
        if (this.status != UserStatus.ACTIVE) {
            throw new IllegalStateException(
                "只有正常状态的用户才能停机");
        }
        
        if (reason == SuspendReason.ARREARS) {
            this.status = UserStatus.SUSPENDED_ARREARS;
        } else if (reason == SuspendReason.USER_REQUEST) {
            this.status = UserStatus.SUSPENDED_REPORT;
        }
        
        // 发布领域事件
        this.addDomainEvent(new UserStatusChangedEvent(
            this.userId, UserStatus.ACTIVE, this.status, reason));
    }
    
    // 复机
    public void resume() {
        if (this.status != UserStatus.SUSPENDED_ARREARS 
            && this.status != UserStatus.SUSPENDED_REPORT) {
            throw new IllegalStateException(
                "只有停机状态的用户才能复机");
        }
        
        // 欠费停机需要先检查欠费是否结清
        if (this.status == UserStatus.SUSPENDED_ARREARS) {
            if (!this.isArrearsClear()) {
                throw new BusinessException(
                    "欠费未结清，不能复机");
            }
        }
        
        UserStatus oldStatus = this.status;
        this.status = UserStatus.ACTIVE;
        
        // 发布领域事件
        this.addDomainEvent(new UserStatusChangedEvent(
            this.userId, oldStatus, this.status, null));
    }
    
    // 检查欠费是否结清
    private boolean isArrearsClear() {
        // 调用账户上下文检查欠费
        return accountService.checkArrears(this.customerId);
    }
}
```

### 8.2 状态转换服务

```java
@Service
public class UserStateTransitionService {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private EventPublisher eventPublisher;
    
    @Transactional
    public void activateUser(UserId userId) {
        // 加载用户聚合（乐观锁）
        User user = userRepository.findById(userId);
        
        // 执行状态转换
        user.activate();
        
        // 保存用户聚合（乐观锁会自动检查版本）
        userRepository.save(user);
        
        // 发布领域事件
        user.getDomainEvents().forEach(event -> 
            eventPublisher.publish(event));
        
        // 清空领域事件
        user.clearDomainEvents();
    }
    
    @Transactional
    public void suspendUser(UserId userId, SuspendReason reason) {
        User user = userRepository.findById(userId);
        user.suspend(reason);
        userRepository.save(user);
        
        user.getDomainEvents().forEach(event -> 
            eventPublisher.publish(event));
        user.clearDomainEvents();
    }
}
```

## 九、总结

本文档详细定义了客户中心核心实体的状态机，包括：

1. **客户状态机**：4个状态，涵盖从开户到销户的完整生命周期
2. **用户状态机**：6个状态，是最复杂的状态机，占业务量90%
3. **SIM卡状态机**：4个状态，管理SIM卡的生命周期
4. **账户状态机**：3个状态，管理账户的使用状态
5. **产品订购状态机**：5个状态，管理产品订购的生命周期

状态机设计的关键点：
- **状态定义清晰**：每个状态有明确的业务含义
- **转换规则明确**：定义了合法的状态转换路径和条件
- **事件驱动**：状态转换发布领域事件，实现解耦
- **并发控制**：使用乐观锁防止并发冲突
- **异常处理**：非法转换抛出异常，保证数据一致性

这些状态机是系统核心业务逻辑的基础，必须严格遵守和实现。
