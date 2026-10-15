# 合同 / 订单 API 字段契约

> 前后端 JSON 字段名必须与本表一致。  
> **与作业管理（homework）共用的字段，命名必须与作业一致**，便于后续任务管理改造时订单→任务直接继承。

## 字段对齐原则

| 场景 | 规则 |
| --- | --- |
| 作业已有字段 | 合同/订单 **沿用作业字段名**（如 `orderType`、`quantityDelivered`） |
| 合同/订单独有 | 使用本域命名（如 `contractNo`、`orderNo`） |
| 任务管理 | **本期不改**；后续基于作业改造时从订单继承上述共用字段 |

### 与作业管理（Homework / HomeworkConfiguration）对齐

| 作业字段 | 订单字段 | 说明 |
| --- | --- | --- |
| `orderType` | `orderType` | 1条件筛选匹配 2指定公司名单 |
| `quantityDelivered` | `quantityDelivered` | 交付数量 |
| `deliveredCount` | `deliveredCount` | 已交付数量 |
| `homeworkUrl` | `homeworkUrl` | 指定名单附件 URL |
| `homeworkUrlName` | `homeworkUrlName` | 指定名单附件名 |
| `customerName` | `customerName` | 客户名称（列表筛选同作业） |
| `tenantId` / `tenantName` | `tenantId` / `tenantName` | 租户 |
| `businessLine` | `businessLine` | 业务线 |
| `reachLevel` | `reachLevel` | 触达程度 |
| `companyName` / `creditCode` | `companies[]` | 指定名单公司行 |

## 合同 Contract

（合同域字段无作业等价项，保持独立命名）

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| contractNo | string | CT+日期+序号 |
| contractName | string | |
| tenantId / tenantName | string | |
| services[].phoneQuantity | number | 含手机服务项 |
| services[].leadsQuantity | number | 默认带入订单 quantityDelivered |

## 订单 Order

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| orderNo | string | ORD+日期+序号 |
| contractId / contractNo | | |
| tenantId / tenantName / customerName | | customerName 与作业一致 |
| orderType | number | **同作业** 1/2 |
| orderTypeStr | string | |
| quantityDelivered | number | **同作业** |
| deliveredCount | number | **同作业** |
| homeworkUrl / homeworkUrlName | string | **同作业** |
| listRowCount | number | 名单条数 N（任务校验用，作业暂无列） |
| companyConditions / contactConditions | FilterCondition[] | 条件构建器 |
| deliveryCycle / deliveryFrequency / frequencyRule / deliveryCycleQuantity | | 交付配置（订单/任务域扩展） |
| guideQuota | number | 导迹额度（合同/订单域） |

## 接口

不变：`/homeworkJob/v1/contracts/*`、`/homeworkJob/v1/orders/*`
