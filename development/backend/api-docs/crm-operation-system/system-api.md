# 系统管理应用 - API接口文档

## 概述

本文档描述了CRM运营平台系统管理应用的所有API接口。

**基础路径**: `http://localhost:18101`

**统一响应格式**:

成功响应：
```json
{
  "code": 200,
  "message": "操作成功",
  "data": { ... },
  "timestamp": 1728893400000
}
```

错误响应：
```json
{
  "code": 404,
  "message": "请求的资源不存在：v1/user/login",
  "data": null,
  "timestamp": 1728893400000
}
```

## 1. 认证接口

### 1.1 用户登录

**接口地址**：`POST /v1/auth/login`

**请求参数**：

```json
{
  "username": "admin",
  "password": "admin123"
}
```

**响应示例**：

```json
{
  "code": 200,
  "message": "操作成功",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "userInfo": {
      "id": 1,
      "username": "admin",
      "realName": "系统管理员",
      "email": "admin@agtnovo.com",
      "mobile": "13800138000",
      "avatar": null,
      "status": 1,
      "statusName": "启用",
      "lastLoginTime": "2025-10-14 13:00:00",
      "lastLoginIp": "127.0.0.1",
      "createTime": "2025-10-14 10:00:00",
      "updateTime": "2025-10-14 13:00:00"
    }
  },
  "timestamp": 1728893400000
}
```

### 1.2 用户登出

**接口地址**：`POST /v1/auth/logout`

**请求头**：
```
Authorization: Bearer {token}
```

**响应示例**：

```json
{
  "code": 200,
  "message": "操作成功",
  "data": null,
  "timestamp": 1728893400000
}
```

## 2. 用户管理接口

### 2.1 创建用户

**接口地址**：`POST /v1/users`

**请求头**：
```
Authorization: Bearer {token}
```

**请求参数**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| username | String | 是 | 登录用户名，长度4-50，支持字母、数字和特殊字符`.-_@` |
| password | String | 是 | 密码，长度6-20 |
| realName | String | 是 | 真实姓名，最大长度50 |
| email | String | 否 | 邮箱，最大长度100 |
| mobile | String | 否 | 手机号，11位数字，1开头 |
| avatar | String | 否 | 头像URL，最大长度255 |

**请求示例**：

```json
{
  "username": "test.user@company",
  "password": "123456",
  "realName": "测试用户",
  "email": "test@agtnovo.com",
  "mobile": "13900139000",
  "avatar": "https://example.com/avatar.jpg"
}
```

**响应示例**：

```json
{
  "code": 200,
  "message": "操作成功",
  "data": {
    "id": 2,
    "username": "testuser",
    "realName": "测试用户",
    "email": "test@agtnovo.com",
    "mobile": "13900139000",
    "avatar": "https://example.com/avatar.jpg",
    "status": 1,
    "statusName": "启用",
    "lastLoginTime": null,
    "lastLoginIp": null,
    "createTime": "2025-10-14 13:00:00",
    "updateTime": "2025-10-14 13:00:00"
  },
  "timestamp": 1728893400000
}
```

### 2.2 更新用户

**接口地址**：`PUT /v1/users/{id}`

**请求头**：
```
Authorization: Bearer {token}
```

**请求参数**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| password | String | 否 | 密码，长度6-20（不传或传空则不修改） |
| realName | String | 否 | 真实姓名，最大长度50 |
| email | String | 否 | 邮箱，最大长度100 |
| mobile | String | 否 | 手机号，11位数字，1开头 |
| avatar | String | 否 | 头像URL，最大长度255 |

**说明**：
- 用户名（username）不可修改
- 所有字段均为可选，仅更新传入的字段
- 支持部分更新

**请求示例**：

```json
{
  "password": "new123456",
  "realName": "测试用户（已修改）",
  "email": "test@agtnovo.com",
  "mobile": "13900139000",
  "avatar": "https://example.com/avatar.jpg"
}
```

**响应示例**：

```json
{
  "code": 200,
  "message": "操作成功",
  "data": null,
  "timestamp": 1728893400000
}
```

### 2.3 删除用户

**接口地址**：`DELETE /v1/users/{id}`

**请求头**：
```
Authorization: Bearer {token}
```

**响应示例**：

```json
{
  "code": 200,
  "message": "操作成功",
  "data": null,
  "timestamp": 1728893400000
}
```

### 2.4 查询用户详情

**接口地址**：`GET /v1/users/{id}`

**请求头**：
```
Authorization: Bearer {token}
```

**响应示例**：

```json
{
  "code": 200,
  "message": "操作成功",
  "data": {
    "id": 1,
    "username": "admin",
    "realName": "系统管理员",
    "email": "admin@agtnovo.com",
    "mobile": "13800138000",
    "avatar": null,
    "status": 1,
    "statusName": "启用",
    "lastLoginTime": "2025-10-14 13:00:00",
    "lastLoginIp": "127.0.0.1",
    "createTime": "2025-10-14 10:00:00",
    "updateTime": "2025-10-14 13:00:00"
  },
  "timestamp": 1728893400000
}
```

### 2.5 分页查询用户列表

**接口地址**：`GET /v1/users`

**请求头**：
```
Authorization: Bearer {token}
```

**请求参数**：

| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| username | String | 否 | 用户名（模糊查询） |
| realName | String | 否 | 真实姓名（模糊查询） |
| mobile | String | 否 | 手机号（模糊查询） |
| status | Integer | 否 | 状态：1-启用，0-禁用 |
| pageNum | Integer | 否 | 页码，默认1 |
| pageSize | Integer | 否 | 每页大小，默认10 |

**响应示例**：

```json
{
  "code": 200,
  "message": "操作成功",
  "data": {
    "records": [
      {
        "id": 1,
        "username": "admin",
        "realName": "系统管理员",
        "email": "admin@agtnovo.com",
        "mobile": "13800138000",
        "avatar": null,
        "status": 1,
        "statusName": "启用",
        "lastLoginTime": "2025-10-14 13:00:00",
        "lastLoginIp": "127.0.0.1",
        "createTime": "2025-10-14 10:00:00",
        "updateTime": "2025-10-14 13:00:00"
      }
    ],
    "total": 1,
    "current": 1,
    "size": 10
  },
  "timestamp": 1728893400000
}
```

### 2.6 启用用户

**接口地址**：`PUT /v1/users/{id}/enable`

**请求头**：
```
Authorization: Bearer {token}
```

**响应示例**：

```json
{
  "code": 200,
  "message": "用户已启用",
  "data": null,
  "timestamp": 1728893400000
}
```

### 2.7 禁用用户

**接口地址**：`PUT /v1/users/{id}/disable`

**请求头**：
```
Authorization: Bearer {token}
```

**响应示例**：

```json
{
  "code": 200,
  "message": "用户已禁用",
  "data": null,
  "timestamp": 1728893400000
}
```

## 3. 错误码说明

| 错误码 | 说明 |
|--------|------|
| 200 | 操作成功 |
| 400 | 参数校验失败 |
| 401 | 未授权（未登录或Token无效） |
| 403 | 禁止访问（账号已禁用） |
| 404 | 资源不存在（接口不存在或数据不存在） |
| 500 | 系统异常 |

## 4. 注意事项

1. 除登录接口外，所有接口都需要在请求头中携带Token
2. Token格式：`Authorization: Bearer {token}`
3. Token有效期：开发环境7天，生产环境1天
4. 密码使用BCrypt加密存储
5. 用户删除为逻辑删除，不会真正删除数据
6. **用户名规则**：
   - 长度：4-50个字符
   - 支持字符：字母（a-z, A-Z）、数字（0-9）、特殊字符（`.`、`-`、`_`、`@`）
   - 示例：`admin`、`user.name`、`test-user`、`user_123`、`user@company`
   - 用户名创建后不可修改

