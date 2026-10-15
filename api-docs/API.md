# CRM 中台坐席与供应商相关接口文档

基础路径说明：接口均以 `/v1` 为前缀，统一返回格式 `Result<T>`（含 code、message、data、timestamp）。

---

## 1. PlatformSeatController - 中台坐席

**基础路径：** `/v1/platform-seat`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /v1/platform-seat/page | 分页查询中台坐席（仅 is_deleted=0） |
| GET | /v1/platform-seat/{id} | 根据 ID 查询中台坐席 |
| POST | /v1/platform-seat | 新增中台坐席（未传 seatNo 时自动生成 S0001、S0002…） |
| PUT | /v1/platform-seat/{id} | 修改中台坐席 |
| PUT | /v1/platform-seat/{id}/status | 单条启用/禁用（0-禁用，1-启用） |
| DELETE | /v1/platform-seat/{id} | 单条逻辑删除 |
| POST | /v1/platform-seat/batch-status | 批量启用/禁用 |
| POST | /v1/platform-seat/batch-delete | 批量逻辑删除 |
| GET | /v1/platform-seat/resource-mapping | 中台坐席映射详情（当前鉴权 + 历史鉴权列表） |

### 1.1 分页查询 - GET /v1/platform-seat/page

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| pageNo | long | 否 | 页码，默认 1 |
| pageSize | long | 否 | 每页条数，默认 10 |
| seatNo | String | 否 | 坐席编号，模糊 |
| seatName | String | 否 | 坐席名称，模糊 |
| status | Integer | 否 | 启用状态：0-未启用，1-启用 |
| authenticationStatus | Integer | 否 | 授权状态：0-未授权，1-已授权 |

### 1.2 根据 ID 查询 - GET /v1/platform-seat/{id}

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | Long | 是 | 路径参数，中台坐席 ID |

### 1.3 新增 - POST /v1/platform-seat

请求体：`PlatformSeat`（JSON），含 seatName、description、status、authenticationStatus 等。

- **seatNo**：可选。不传或传空时由后端自动生成坐席编号，规则为以 `S` 开头、后接 4 位序号（如 S0001、S0002），取表内同前缀最大序号加 1。

### 1.4 修改 - PUT /v1/platform-seat/{id}

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | Long | 是 | 路径参数 |
请求体：`PlatformSeat`（JSON）。

### 1.5 单条启用/禁用 - PUT /v1/platform-seat/{id}/status

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | Long | 是 | 路径参数 |
| status | Integer | 是 | 查询参数，0-禁用，1-启用 |

### 1.6 单条删除 - DELETE /v1/platform-seat/{id}

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | Long | 是 | 路径参数 |

### 1.7 批量启用/禁用 - POST /v1/platform-seat/batch-status

请求体示例：

```json
{
  "ids": [ 1, 2, 3 ],
  "status": 1
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| ids | List\<Long\> | 是 | 坐席 ID 列表 |
| status | Integer | 是 | 0-禁用，1-启用 |

### 1.8 批量删除 - POST /v1/platform-seat/batch-delete

请求体：`List<Long>`，如 `[ 1, 2, 3 ]`。

### 1.9 中台坐席映射详情（资源鉴权）- GET /v1/platform-seat/resource-mapping

用于中台坐席详情页「映射信息 / 分配历史」：根据当前有效 `resource_seat` 解析当前鉴权，并返回历史分配对应的鉴权列表。

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| platformSeatId | Long | 是 | 中台坐席 ID |

返回：`PlatformSeatResourceMappingVO`

| 字段 | 类型 | 说明 |
|------|------|------|
| currentResourceAuthentication | ResourceAuthentication | 当前鉴权：最新一条 `resource_seat`（`platform_seat_id` 匹配且 `is_deleted=0`，按 `id` desc 取 1）的 `resource_id` 对应鉴权；无关联时为 null |
| historyList | List\<ResourceAuthentication\> | 分配历史：同坐席下 `resource_seat_history`（`is_deleted=0`，按 `create_time` desc）逐条按 `resource_id` 解析的鉴权；查不到的记录跳过 |

---

## 2. PlatformSupplierSeatController - 中台坐席与供应商坐席关联

**基础路径：** `/v1/platform-supplier-seat`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /v1/platform-supplier-seat/getPlatformSupplierList | 按中台坐席分页查询当前关联列表（含供应商名称、坐席账号、合同时间） |
| POST | /v1/platform-supplier-seat/saveAssociation | 关联保存（先写取消关联 history，再 is_deleted=1，再新增） |
| GET | /v1/platform-supplier-seat/getAssociationData | 关联数据：可选坐席列表 + 已关联坐席列表 |
| GET | /v1/platform-supplier-seat/page | 分页查询关联表（仅 is_deleted=0） |
| GET | /v1/platform-supplier-seat/{id} | 根据 ID 查询关联 |
| POST | /v1/platform-supplier-seat | 新增关联 |
| PUT | /v1/platform-supplier-seat/{id} | 修改关联 |
| DELETE | /v1/platform-supplier-seat/{id} | 逻辑删除关联 |
| PUT | /v1/platform-supplier-seat/{id}/disassociate | 取消关联并写入 history |

### 2.1 按中台坐席分页查询当前关联 - GET /v1/platform-supplier-seat/getPlatformSupplierList

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| platformSeatId | Long | 是 | 中台坐席 ID |
| pageNo | long | 否 | 默认 1 |
| pageSize | long | 否 | 默认 10 |

返回：`PageResult<PlatformSupplierSeatDetailVO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| id | Long | 关联表主键 |
| platformSeatId | Long | 中台坐席 ID |
| supplierSeatId | Long | 供应商坐席 ID |
| supplierSeatAccount | String | 坐席账号（`supplier_seat.user_account`） |
| supplierId | Long | 供应商 ID |
| supplierName | String | 供应商名称 |
| supplierType | Integer | 坐席平台：1-七鱼，2-涓涌，3-优音 |
| contractStartTime | LocalDate | 合同开始时间 |
| contractEndTime | LocalDate | 合同结束时间 |

### 2.2 关联保存 - POST /v1/platform-supplier-seat/saveAssociation

请求体示例：

```json
{
  "platformSeatId": 100,
  "supplierSeatIds": [ 201, 202, 203 ]
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| platformSeatId | Long | 是 | 中台坐席 ID |
| supplierSeatIds | List\<Long\> | 否 | 本次要关联的供应商坐席 ID 列表（已选择） |

### 2.3 关联数据（双 list）- GET /v1/platform-supplier-seat/getAssociationData

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| platformSeatId | Long | 否 | 中台坐席 ID；不传则 `associatedSupplierSeats` 为空 |

返回：`PlatformSupplierSeatAssociationVO`

| 字段 | 类型 | 说明 |
|------|------|------|
| allSupplierSeats | List\<SupplierSeatAssociationItemVO\> | 可选供应商坐席 |
| associatedSupplierSeats | List\<SupplierSeatAssociationItemVO\> | 已关联供应商坐席（`platform_supplier_seat` 且 `is_deleted=0`、`status=1`） |

**数据范围：**

- 仅包含 **供应商有效**（`supplier.is_deleted=0`）且 **坐席未删除**（`supplier_seat.is_deleted=0`）的记录。
- `platformSeatId` 有值时，`associatedSupplierSeats` 为当前中台坐席下已关联且满足上述条件的坐席。

**SupplierSeatAssociationItemVO 字段：**

| 字段 | 类型 | 说明 |
|------|------|------|
| id | Long | 供应商坐席 ID |
| supplierId | Long | 供应商 ID |
| phone | String | 员工真实手机号 |
| userAccount | String | 坐席账号 |
| accountName | String | 账号名称（仅七鱼平台有值） |
| supplierType | Integer | 坐席平台：1-七鱼，2-涓涌，3-优音（来自 `supplier.supplier_type`） |
| password | String | 密码 |
| supplierName | String | 供应商名称 |
| supplierNo | String | 供应商编号 |

### 2.4 分页查询关联 - GET /v1/platform-supplier-seat/page

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| pageNo | long | 否 | 默认 1 |
| pageSize | long | 否 | 默认 10 |
| platformSeatId | Long | 否 | 中台坐席 ID |
| supplierSeatId | Long | 否 | 供应商坐席 ID |
| status | Integer | 否 | 0-未关联，1-已关联 |

### 2.5 根据 ID 查询 - GET /v1/platform-supplier-seat/{id}

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | Long | 是 | 路径参数 |

### 2.6 新增关联 - POST /v1/platform-supplier-seat

请求体：`PlatformSupplierSeat`（platformSeatId、supplierSeatId、status 等）。

### 2.7 修改关联 - PUT /v1/platform-supplier-seat/{id}

路径参数 `id`，请求体：`PlatformSupplierSeat`。

### 2.8 逻辑删除关联 - DELETE /v1/platform-supplier-seat/{id}

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | Long | 是 | 路径参数 |

### 2.9 取消关联 - PUT /v1/platform-supplier-seat/{id}/disassociate

将 status 置为 0 并写入 `platform_supplier_seat_history`。路径参数 `id`。

---

## 3. PlatformSupplierSeatHistoryController - 关联历史

**基础路径：** `/v1/platform-supplier-seat-history`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /v1/platform-supplier-seat-history/page | 分页查询关联历史（仅 is_deleted=0） |

### 3.1 分页查询历史 - GET /v1/platform-supplier-seat-history/page

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| pageNo | long | 否 | 默认 1 |
| pageSize | long | 否 | 默认 10 |
| platformSeatId | Long | 否 | 中台坐席 ID |
| supplierSeatId | Long | 否 | 供应商坐席 ID |

返回：`PageResult<PlatformSupplierSeatHistoryVO>`，除历史表字段外含 `supplierSeatAccount`、`supplierId`、`supplierName`、`supplierType`（1-七鱼，2-涓涌，3-优音）。

---

## 4. SupplierApiConfigController - 供应商接口配置

**基础路径：** `/v1/supplier-api-config`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /v1/supplier-api-config/page | 分页查询接口配置（仅 is_deleted=0） |
| GET | /v1/supplier-api-config/{id} | 根据 ID 查询 |
| POST | /v1/supplier-api-config | 新增接口配置 |
| PUT | /v1/supplier-api-config/{id} | 修改接口配置 |
| DELETE | /v1/supplier-api-config/{id} | 逻辑删除 |

### 4.1 分页查询 - GET /v1/supplier-api-config/page

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| pageNo | long | 否 | 默认 1 |
| pageSize | long | 否 | 默认 10 |
| supplierId | Long | 否 | 供应商 ID |
| apiNo | String | 否 | 接口编号，模糊 |
| apiName | String | 否 | 接口名称，模糊 |

### 4.2 根据 ID 查询 - GET /v1/supplier-api-config/{id}

路径参数：`id`（Long）。

### 4.3 新增 - POST /v1/supplier-api-config

请求体：`SupplierApiConfig`（supplierId、apiNo、apiName、apiPath、apiType 等）。

### 4.4 修改 - PUT /v1/supplier-api-config/{id}

路径参数 `id`，请求体：`SupplierApiConfig`。

### 4.5 删除 - DELETE /v1/supplier-api-config/{id}

路径参数：`id`。

---

## 5. SupplierAuthenticationConfigController - 供应商鉴权配置

**基础路径：** `/v1/supplier-authentication-config`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /v1/supplier-authentication-config/page | 分页查询鉴权配置（仅 is_deleted=0） |
| GET | /v1/supplier-authentication-config/{id} | 根据 ID 查询 |
| POST | /v1/supplier-authentication-config | 新增鉴权配置 |
| PUT | /v1/supplier-authentication-config/{id} | 修改鉴权配置 |
| DELETE | /v1/supplier-authentication-config/{id} | 逻辑删除 |

### 5.1 分页查询 - GET /v1/supplier-authentication-config/page

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| pageNo | long | 否 | 默认 1 |
| pageSize | long | 否 | 默认 10 |
| supplierId | Long | 否 | 供应商 ID |
| authenticationName | String | 否 | 鉴权类型，模糊 |

### 5.2 根据 ID 查询 - GET /v1/supplier-authentication-config/{id}

路径参数：`id`。

### 5.3 新增 - POST /v1/supplier-authentication-config

请求体：`SupplierAuthenticationConfig`（supplierId、authenticationName、authenticationValue 等）。

### 5.4 修改 - PUT /v1/supplier-authentication-config/{id}

路径参数 `id`，请求体：`SupplierAuthenticationConfig`。

### 5.5 删除 - DELETE /v1/supplier-authentication-config/{id}

路径参数：`id`。

---

## 6. SupplierController - 供应商

**基础路径：** `/v1/supplier`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /v1/supplier/page | 分页查询供应商（仅 is_deleted=0） |
| GET | /v1/supplier/{id} | 根据 ID 查询供应商 |
| POST | /v1/supplier | 新增供应商（自动生成 supplierNo） |
| PUT | /v1/supplier/{id} | 修改供应商 |
| DELETE | /v1/supplier/{id} | 逻辑删除（is_deleted=1） |

### 6.1 分页查询 - GET /v1/supplier/page

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| pageNo | long | 否 | 默认 1 |
| pageSize | long | 否 | 默认 10 |

### 6.2 根据 ID 查询 - GET /v1/supplier/{id}

路径参数：`id`。

### 6.3 新增 - POST /v1/supplier

请求体：`Supplier`（supplierName、supplierType、contractStartTime、contractEndTime、supplierCost、contactPeople、contactPhone 等），supplierNo 由后端生成。

**supplierType（坐席平台）：** `Integer`，1-七鱼，2-涓涌，3-优音。坐席账号的平台类型以供应商为准，不再在 `supplier_seat` 表单独维护。

### 6.4 修改 - PUT /v1/supplier/{id}

路径参数 `id`，请求体：`Supplier`。

### 6.5 删除 - DELETE /v1/supplier/{id}

路径参数：`id`。

---

## 7. SupplierSeatController - 供应商坐席账号

**基础路径：** `/v1/supplier-seat`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /v1/supplier-seat/page | 分页查询坐席账号（仅 is_deleted=0，含 supplierTypeName） |
| GET | /v1/supplier-seat/{id} | 根据 ID 查询坐席账号 |
| POST | /v1/supplier-seat | 新增坐席账号 |
| PUT | /v1/supplier-seat/{id} | 修改坐席账号 |
| DELETE | /v1/supplier-seat/{id} | 逻辑删除 |

### 7.1 分页查询 - GET /v1/supplier-seat/page

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| pageNo | long | 否 | 默认 1 |
| pageSize | long | 否 | 默认 10 |
| supplierId | Long | 否 | 供应商 ID |

返回：`PageResult<SupplierSeatVO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| id | Long | 主键 |
| supplierId | Long | 供应商 ID |
| phone | String | 员工真实手机号 |
| userAccount | String | 坐席账号 |
| accountName | String | 账号名称（仅七鱼平台有值） |
| supplierType | Integer | 坐席平台：1-七鱼，2-涓涌，3-优音（来自关联供应商 `supplier_type`） |
| supplierTypeName | String | 平台名称（七鱼/涓涌/优音） |
| password | String | 密码 |
| isDeleted | Integer | 0-有效，1-已删除 |
| createBy | String | 创建人 |
| createTime | LocalDateTime | 创建时间 |
| updateBy | String | 更新人 |
| updateTime | LocalDateTime | 更新时间 |

### 7.2 根据 ID 查询 - GET /v1/supplier-seat/{id}

路径参数：`id`。返回：`SupplierSeat`（字段同下表）。

### 7.3 SupplierSeat 请求体字段

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| supplierId | Long | 是* | 供应商 ID（新增/修改校验必填） |
| phone | String | 否 | 员工真实手机号 |
| userAccount | String | 条件 | 坐席账号；七鱼、优音平台必填 |
| accountName | String | 否 | 账号名称；仅七鱼平台可维护，其他平台保存时置空 |
| password | String | 条件 | 密码；优音平台必填 |
| isDeleted | Integer | 否 | 0-有效，1-已删除 |

坐席平台类型取自关联供应商的 `supplier_type`，**不在** `SupplierSeat` 请求体中传递。

**平台校验（新增、修改均适用，按供应商 `supplier_type` 判断）：**

| supplierType | 说明 | 坐席必填字段 |
|--------------|------|----------------|
| 1 | 七鱼 | `userAccount`；可选 `accountName` |
| 3 | 优音 | `userAccount`、`password` |

**唯一性：** 同一 `supplierId` 下 `userAccount` 在 `is_deleted=0` 记录中不可重复（修改时排除自身）。

修改时若请求体未传某字段，校验前会用库中原值补全（如仅改 `phone` 时不强制重传密码）。

### 7.4 新增 - POST /v1/supplier-seat

请求体：`SupplierSeat`（JSON），校验失败返回 `400`。

### 7.5 修改 - PUT /v1/supplier-seat/{id}

路径参数 `id`，请求体：`SupplierSeat`（JSON），校验规则同新增；记录不存在返回 `404`。

### 7.6 删除 - DELETE /v1/supplier-seat/{id}

路径参数：`id`。

---

## 8. SupplierSeatPhoneController - 供应商坐席电话

**基础路径：** `/v1/supplier-seat-phone`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /v1/supplier-seat-phone/page | 分页查询坐席电话（仅 is_deleted=0） |
| POST | /v1/supplier-seat-phone/save | 批量保存坐席电话（list 中无 id 新增，有 id 编辑） |
| GET | /v1/supplier-seat-phone/{id} | 根据 ID 查询坐席电话 |
| POST | /v1/supplier-seat-phone | 新增坐席电话 |
| PUT | /v1/supplier-seat-phone/{id} | 修改坐席电话 |
| DELETE | /v1/supplier-seat-phone/{id} | 逻辑删除 |

### 8.1 分页查询 - GET /v1/supplier-seat-phone/page

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| pageNo | long | 否 | 默认 1 |
| pageSize | long | 否 | 默认 10 |
| supplierSeatId | Long | 否 | 坐席 ID |

### 8.2 批量保存 - POST /v1/supplier-seat-phone/save

请求体示例：

```json
{
  "supplierSeatId": 123,
  "items": [
    { "id": null, "phone": "17382932333" },
    { "id": 456, "phone": "17382932334" }
  ]
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| supplierSeatId | Long | 是 | 坐席 ID |
| items | List\<SupplierSeatPhoneSaveItem\> | 是 | 每项 id 为空则新增，有值则编辑；phone 必填 |

### 8.3 根据 ID 查询 - GET /v1/supplier-seat-phone/{id}

路径参数：`id`。

### 8.4 新增 - POST /v1/supplier-seat-phone

请求体：`SupplierSeatPhone`（supplierSeatId、phone 等）。

### 8.5 修改 - PUT /v1/supplier-seat-phone/{id}

路径参数 `id`，请求体：`SupplierSeatPhone`。

### 8.6 删除 - DELETE /v1/supplier-seat-phone/{id}

路径参数：`id`。

---

## 9. ResourceAuthenticationController - 资源授权

**基础路径：** `/v1/resource-authentication`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /v1/resource-authentication/page | 分页查询资源授权（仅 is_deleted=0） |
| GET | /v1/resource-authentication/{id} | 根据 ID 查询资源授权 |
| POST | /v1/resource-authentication | 新增资源授权（未传 appKey/appSecret 时自动生成，返回保存后的对象） |
| PUT | /v1/resource-authentication/{id} | 修改资源授权 |
| PUT | /v1/resource-authentication/{id}/status | 单条启用/禁用（status：0-失效，1-生效） |
| POST | /v1/resource-authentication/batch-status | 批量启用/禁用 |
| PUT | /v1/resource-authentication/{id}/reset-secret | 重置 AppSecret（重新生成并保存） |
| DELETE | /v1/resource-authentication/{id} | 逻辑删除（is_deleted=1） |

### 9.1 分页查询 - GET /v1/resource-authentication/page

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| pageNo | long | 否 | 默认 1 |
| pageSize | long | 否 | 默认 10 |
| authenticationName | String | 否 | 授权名称，模糊查询 |
| status | Integer | 否 | 状态：0-失效，1-生效 |

### 9.2 根据 ID 查询 - GET /v1/resource-authentication/{id}

路径参数：`id`（Long）。

### 9.3 新增 - POST /v1/resource-authentication

请求体：`ResourceAuthentication`（JSON），建议字段：

- `authenticationName`：授权名称
- `appKey`：可选；未传或为空时后端自动生成，规则：前缀 `AK` + 16 位大写字母数字
- `appSecret`：可选；未传或为空时后端自动生成，规则：32 字节强随机数转换为 64 位十六进制字符串
- `contactPeople`：联系人姓名
- `contactEmail`：联系人邮箱
- `yyCallbackUrl`：优音回调地址
- `qyCallbackUrl`：七鱼回调地址
- `status`：状态（0-失效，1-生效）
- `startDate`：授权生效日期（yyyy-MM-dd）
- `endDate`：授权失效日期（yyyy-MM-dd）

返回：`ResourceAuthentication`（保存后的完整对象，含后端生成的 `id`、`appKey`、`appSecret` 等）。

### 9.4 修改 - PUT /v1/resource-authentication/{id}

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | Long | 是 | 路径参数 |

请求体：`ResourceAuthentication`（JSON），可修改授权名称、联系人信息、优音/七鱼回调地址、有效期、状态等。

### 9.5 单条启用/禁用 - PUT /v1/resource-authentication/{id}/status

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | Long | 是 | 路径参数 |
| status | Integer | 是 | 查询参数，0-失效，1-生效 |

仅对 `is_deleted=0` 的记录生效。

### 9.6 批量启用/禁用 - POST /v1/resource-authentication/batch-status

请求体示例：

```json
{
  "ids": [1, 2, 3],
  "status": 1
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| ids | List\<Long\> | 是 | 资源授权 ID 列表 |
| status | Integer | 是 | 0-失效，1-生效 |

仅对 `is_deleted=0` 的记录生效。

### 9.7 重置 AppSecret - PUT /v1/resource-authentication/{id}/reset-secret

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | Long | 是 | 路径参数 |

说明：重新生成一条记录的 `appSecret` 并保存，原 secret 作废。仅对存在且未逻辑删除的记录生效。

### 9.8 删除 - DELETE /v1/resource-authentication/{id}

路径参数：`id`，逻辑删除（`is_deleted` 置为 1）。

---

## 10. ResourceSeatController - 资源中台坐席

**基础路径：** `/v1/resource-seat`

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /v1/resource-seat/getPlatformSeatAssociationData | 分配中台坐席弹窗数据（可选坐席 + 已关联坐席，不分页） |
| POST | /v1/resource-seat/savePlatformSeatAssociation | 保存资源与中台坐席关联（写历史、更新 platform_seat 状态与授权状态） |
| PUT | /v1/resource-seat/release | 释放中台坐席（逻辑删除关联、写历史、恢复未授权） |
| GET | /v1/resource-seat/page | 分页查询资源中台坐席（仅 is_deleted=0） |
| GET | /v1/resource-seat/getPlatformSeatList | 某资源下已分配坐席列表（含坐席名称、供应商坐席数） |
| GET | /v1/resource-seat/{id} | 根据 ID 查询 |
| POST | /v1/resource-seat | 新增 |
| PUT | /v1/resource-seat/{id} | 修改 |
| DELETE | /v1/resource-seat/{id} | 逻辑删除 |

### 10.1 分配坐席弹窗数据 - GET /v1/resource-seat/getPlatformSeatAssociationData

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| resourceId | Long | 否 | 资源（鉴权）ID；不传则「已关联」列表为空 |

返回：`ResourceSeatPlatformAssociationVO`

| 字段 | 类型 | 说明 |
|------|------|------|
| allPlatformSeats | List\<PlatformSeat\> | 可选中台坐席：`platform_seat` 满足 `is_deleted=0`、`status=1`、`authentication_status=0`，且在 `platform_supplier_seat` 中存在有效关联（`is_deleted=0` 且 `status=1`），不分页 |
| associatedPlatformSeats | List\<PlatformSeat\> | 当前 `resourceId` 下已关联的中台坐席（`resource_seat` 且 `is_deleted=0`） |

### 10.2 保存资源与中台坐席关联 - POST /v1/resource-seat/savePlatformSeatAssociation

请求体示例：

```json
{
  "resourceId": 100,
  "platformSeatIds": [ 501, 502 ]
}
```

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| resourceId | Long | 是 | 资源（鉴权）ID |
| platformSeatIds | List\<Long\> | 否 | 本次勾选保留的中台坐席 ID；不传或空则清空该资源下全部关联 |

说明：相对保存前被去掉勾选的关联会写入 `resource_seat_history`（`association_time` 取原 `resource_seat.create_time`）；再将该 `resource_id` 下原有 `resource_seat` 全部逻辑删除后按列表重建；对已勾选坐席将 `platform_seat` 更新为启用且已授权；对去掉勾选且全局无其它绑定的坐席将授权状态恢复为未授权。

### 10.3 释放中台坐席 - PUT /v1/resource-seat/release

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| resourceId | Long | 是 | 资源（鉴权）ID |
| platformSeatId | Long | 是 | 中台坐席 ID |

说明：按 `resource_id` + `platform_seat_id` 查找 `is_deleted=0` 的 `resource_seat`；不存在则返回 404。成功时：

1. 写入 `resource_seat_history`（`association_time` 取原 `resource_seat.create_time`）
2. 将对应 `resource_seat` 的 `is_deleted` 置为 1
3. 将对应 `platform_seat` 的 `authentication_status` 置为 0（未授权）

返回：`Result<Boolean>`，`data` 为 `true` 表示成功。

### 10.4 分页查询 - GET /v1/resource-seat/page

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| pageNo | long | 否 | 默认 1 |
| pageSize | long | 否 | 默认 10 |
| resourceId | Long | 否 | 资源 ID |
| platformSeatId | Long | 否 | 中台坐席 ID |

### 10.5 某资源已分配坐席列表 - GET /v1/resource-seat/getPlatformSeatList

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| resourceId | Long | 是 | 资源 ID |
| pageNo | long | 否 | 默认 1 |
| pageSize | long | 否 | 默认 10 |

返回：`PageResult<ResourceSeatDetailVO>`（含 seatNo、seatName、supplierSeatCount 等）。

### 10.6 根据 ID 查询 - GET /v1/resource-seat/{id}

路径参数：`id`。

### 10.7 新增 - POST /v1/resource-seat

请求体：`ResourceSeat`（resourceId、platformSeatId 等）。

### 10.8 修改 - PUT /v1/resource-seat/{id}

路径参数 `id`，请求体：`ResourceSeat`。

### 10.9 删除 - DELETE /v1/resource-seat/{id}

路径参数：`id`，逻辑删除（`is_deleted=1`）。

---

## 11. ResourceAuthenticationFeignController - 资源鉴权 Feign 接口

**基础路径：** `/v1/resource-authentication`

供其它微服务通过 OpenFeign 调用。Feign 客户端见 `ResourceAuthenticationFeignClient`（`name = crm-cc-platform-core`）。

| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /v1/resource-authentication/supplier-seat/page | 按 appKey 校验鉴权后，分页返回绑定的供应商坐席账号 |
| GET | /v1/resource-authentication/supplier-seat/count | 按 appKey 校验鉴权后，统计绑定的供应商坐席账号总数 |

### 11.1 分页查询绑定的供应商坐席 - GET /v1/resource-authentication/supplier-seat/page

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| appKey | String | 是 | 资源鉴权 AppKey |
| pageNo | long | 否 | 页码，默认 1 |
| pageSize | long | 否 | 每页条数，默认 10 |

**鉴权条件：** `resource_authentication` 中 `app_key` 匹配，且 `is_deleted=0`、`status=1`。

**关联路径：** `resource_seat`（`resource_id` = 鉴权 ID，`is_deleted=0`）→ `platform_supplier_seat`（`is_deleted=0`、`status=1`）→ `supplier_seat`（`is_deleted=0`）。

返回：`PageResult<ResourceAuthSupplierSeatPageVO>`

| 字段 | 类型 | 说明 |
|------|------|------|
| id | Long | platform_supplier_seat 主键 ID |
| platformSeatId | Long | 中台坐席 ID |
| seatNo | String | 中台坐席编号 |
| seatName | String | 中台坐席名称 |
| supplierSeatId | Long | 供应商坐席 ID |
| supplierId | Long | 供应商 ID |
| phone | String | 员工真实手机号 |
| userAccount | String | 供应商坐席账号 |
| accountName | String | 账号名称（仅七鱼平台有值） |
| password | String | 供应商坐席密码 |
| supplierType | Integer | 坐席平台：1-七鱼，2-涓涌，3-优音 |
| supplierTypeName | String | 平台名称 |

鉴权失败返回 `401`，`message`：`appKey 无效，或资源鉴权未生效`。

### 11.2 统计绑定的供应商坐席总数 - GET /v1/resource-authentication/supplier-seat/count

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| appKey | String | 是 | 资源鉴权 AppKey |

**鉴权条件与关联路径：** 同 11.1。

返回：`Result<Long>`，`data` 为 `supplier_seat` 有效记录总数（`is_deleted=0`）；无绑定坐席时返回 `0`。鉴权失败返回 `401`。

---

## 12. AutoCallRecordController - 自动外呼记录

**基础路径：** `/v1/auto-call-record`

中台统一发起优音 IVR 自动外呼：校验 `appKey` → 按 `dnis` 逐条落库 → 批量调用优音 OpenAPI → 按返回 `data.dnis` 回写 `sessionId`。

| 方法 | 路径 | 说明 |
|------|------|------|
| POST | /v1/auto-call-record/push | 发起优音自动外呼 |

### 12.1 发起自动外呼 - POST /v1/auto-call-record/push

请求体：`AutoCallRecordPushRequest`（JSON）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| appKey | String | 是 | 来源 AppKey（`resource_authentication.app_key`，需生效） |
| items | List\<AutoCallRecordPushItem\> | 是 | 外呼明细列表 |

**AutoCallRecordPushItem：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| companyId | Long | 是 | 公司 ID |
| contactId | Long | 是 | 联系人 ID |
| mobile | String | 是 | 被叫手机号 |

**处理逻辑：** 按 `items` 逐条落库 → 汇总 `mobile` 为 `dnis` 批量调用优音 → 按返回 `data.dnis` 回写 `sessionId`（不去重）。

**请求示例：**

```json
{
  "appKey": "AKxxxxxxxx",
  "items": [
    { "companyId": 2061702485289639937, "contactId": 2, "mobile": "15114033183" },
    { "companyId": 2061702485289639937, "contactId": 3, "mobile": "13800138000" }
  ]
}
```

返回：`List<AutoCallRecordVO>`（每个号码一条记录）

| 字段 | 类型 | 说明 |
|------|------|------|
| id | Long | 自动外呼记录主键 |
| companyId | Long | 公司 ID |
| contactId | Long | 联系人 ID |
| mobile | String | 被叫手机号 |
| sessionId | String | 优音会话 ID（外呼成功后回写，对应回调 `id`） |
| appKey | String | 来源 AppKey |
| createTime | LocalDateTime | 创建时间 |

**优音配置（Nacos / application.yml）：** `youyin.tenant-id`、`youyin.secret`、`youyin.wav-audio`；accessToken 缓存 Redis key：`crm-cc-platform-core:youyin:access_token`。

---

*文档根据当前 Controller 代码整理，如有变更以代码为准。*
