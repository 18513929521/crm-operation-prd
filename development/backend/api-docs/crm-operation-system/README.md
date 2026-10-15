# 系统管理应用 - API文档

## 概述

系统管理应用（crm-operation-system）是CRM运营平台的核心应用，提供用户管理、认证授权等基础功能。

**服务名**: `crm-operation-system`  
**端口**: `18101`  
**基础路径**: `http://localhost:18101`

## API文档列表

| 文档名称 | 描述 | 链接 |
|---------|------|------|
| 系统管理API | 用户管理、认证授权等客户端接口 | [system-api.md](./system-api.md) |
| 内部接口API | 服务间调用、Token验证等内部接口 | [internal-api.md](./internal-api.md) |

## 快速开始

### 1. 用户登录

```bash
curl -X POST http://localhost:18101/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "admin",
    "password": "123456"
  }'
```

### 2. 获取用户列表

```bash
curl -X GET http://localhost:18101/v1/users \
  -H "Authorization: Bearer {token}"
```

### 3. 创建用户

```bash
curl -X POST http://localhost:18101/v1/users \
  -H "Authorization: Bearer {token}" \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "password": "123456",
    "realName": "测试用户",
    "email": "test@example.com",
    "mobile": "13900139000"
  }'
```

## 接口分类

### 认证授权模块 (AuthController)

**路径前缀**: `/v1/auth`

| 接口 | 方法 | 说明 | 认证 |
|------|------|------|------|
| `/v1/auth/login` | POST | 用户登录 | ❌ |
| `/v1/auth/logout` | POST | 用户登出 | ✅ |
| `/v1/auth/current-user` | GET | 获取当前用户信息 | ✅ |

### 用户管理模块 (UserController)

**路径前缀**: `/v1/users`

| 接口 | 方法 | 说明 | 认证 |
|------|------|------|------|
| `/v1/users` | POST | 创建用户 | ✅ |
| `/v1/users/{id}` | GET | 查询用户详情 | ✅ |
| `/v1/users` | GET | 分页查询用户列表 | ✅ |
| `/v1/users/{id}` | PUT | 更新用户 | ✅ |
| `/v1/users/{id}` | DELETE | 删除用户 | ✅ |
| `/v1/users/{id}/enable` | PUT | 启用用户 | ✅ |
| `/v1/users/{id}/disable` | PUT | 禁用用户 | ✅ |

### 内部接口模块 (InternalController)

**路径前缀**: `/v1/internal`

| 接口 | 方法 | 说明 | 认证 |
|------|------|------|------|
| `/v1/internal/current-user` | GET | 获取当前用户（服务间） | ⚠️ 白名单 |
| `/v1/internal/validate-token` | POST | 验证Token有效性 | ⚠️ 白名单 |

## 认证说明

### Token获取

通过登录接口 `/v1/auth/login` 获取Token：

```json
{
  "code": 200,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiJ9...",
    "userInfo": { ... }
  }
}
```

### Token使用

在需要认证的接口请求头中携带Token：

```
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

### Token有效期

- **开发环境**: 7天
- **测试环境**: 3天
- **生产环境**: 1天

## 测试账号

### 开发环境

| 用户名 | 密码 | 角色 | 说明 |
|--------|------|------|------|
| admin | 123456 | 管理员 | 系统管理员账号 |

⚠️ **注意**: 生产环境部署后请立即修改默认密码！

## 错误处理

### 常见错误

| 错误码 | 说明 | 解决方案 |
|--------|------|---------|
| 400 | 参数校验失败 | 检查请求参数是否符合要求 |
| 401 | Token无效或已过期 | 重新登录获取新Token |
| 403 | 账号已禁用 | 联系管理员启用账号 |
| 404 | 资源不存在 | 检查接口路径和资源ID |
| 500 | 系统异常 | 查看服务端日志，联系技术支持 |

### 错误响应示例

```json
{
  "code": 404,
  "message": "请求的资源不存在：v1/user/login",
  "data": null,
  "timestamp": 1728893400000
}
```

## 版本历史

### v1.0.0 (2025-10-19)
- ✅ 实现用户登录、登出功能
- ✅ 实现用户CRUD管理
- ✅ 实现用户状态管理（启用/禁用）
- ✅ 实现内部服务间接口
- ✅ 添加404错误处理

## 相关链接

- [开发规范](../../crm-operation-system/docs/development/controller-design.md)
- [数据库设计](../../crm-operation-system/sql/schema/)
- [集成测试](../../crm-operation-system/tests/)

---

**文档版本**: v1.0  
**最后更新**: 2025-10-19  
**维护团队**: CRM Team

