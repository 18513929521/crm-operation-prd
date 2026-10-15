# 内部接口API文档

## 概述

内部接口模块提供给其他微服务调用的接口，用于服务间通信和Token验证。

**基础路径**: `/v1/internal`

**认证方式**: 这些接口在JWT拦截器白名单中，不需要经过身份认证拦截器

**功能列表**:
1. 获取当前登录用户信息
2. 验证Token有效性

---

## 接口列表

### 1. 获取当前登录用户信息

**接口说明**: 供其他微服务调用，用于获取当前用户的完整信息

**接口地址**: `GET /v1/internal/current-user`

**请求头**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| Authorization | String | 是 | Bearer Token |

**响应示例**:

成功响应 (200):
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "id": 1,
    "username": "admin",
    "realName": "系统管理员",
    "email": "admin@example.com",
    "mobile": "13800138000",
    "avatar": null,
    "status": 1,
    "statusName": "启用",
    "lastLoginTime": "2025-10-15 14:30:00",
    "lastLoginIp": "127.0.0.1",
    "createTime": "2025-10-14 10:00:00",
    "updateTime": "2025-10-15 14:30:00"
  },
  "timestamp": 1697270400000
}
```

失败响应:
```json
{
  "code": 400,
  "message": "Token不能为空",
  "timestamp": 1697270400000
}
```

```json
{
  "code": 401,
  "message": "Token无效或已过期",
  "timestamp": 1697270400000
}
```

```json
{
  "code": 401,
  "message": "Token已失效，请重新登录",
  "timestamp": 1697270400000
}
```

```json
{
  "code": 404,
  "message": "用户不存在",
  "timestamp": 1697270400000
}
```

```json
{
  "code": 403,
  "message": "账号已禁用，请联系管理员",
  "timestamp": 1697270400000
}
```

---

### 2. 验证Token有效性

**接口说明**: 供网关或其他服务调用，验证Token是否有效并返回用户基本信息

**接口地址**: `POST /v1/internal/validate-token`

**请求头**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| Content-Type | String | 是 | application/json |

**请求参数**:
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| token | String | 是 | 需要验证的Token |

**请求示例**:
```json
{
  "token": "eyJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOjEsInVzZXJuYW1lIjoiYWRtaW4iLCJzdWIiOiJhZG1pbiIsImlhdCI6MTY5NzI3MDQwMCwiZXhwIjoxNjk3MzU2ODAwfQ.xxx"
}
```

**响应示例**:

Token有效 (200):
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "valid": true,
    "userId": 1,
    "username": "admin",
    "realName": "系统管理员",
    "reason": null
  },
  "timestamp": 1697270400000
}
```

Token无效 (200):
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "valid": false,
    "userId": null,
    "username": null,
    "realName": null,
    "reason": "Token无效或已过期"
  },
  "timestamp": 1697270400000
}
```

Token已失效 (200):
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "valid": false,
    "userId": null,
    "username": null,
    "realName": null,
    "reason": "Token已失效"
  },
  "timestamp": 1697270400000
}
```

用户不存在 (200):
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "valid": false,
    "userId": null,
    "username": null,
    "realName": null,
    "reason": "用户不存在"
  },
  "timestamp": 1697270400000
}
```

用户已禁用 (200):
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "valid": false,
    "userId": null,
    "username": null,
    "realName": null,
    "reason": "账号已禁用"
  },
  "timestamp": 1697270400000
}
```

参数校验失败 (400):
```json
{
  "code": 400,
  "message": "Token不能为空",
  "timestamp": 1697270400000
}
```

---

## 错误码说明

| 错误码 | 说明 |
|--------|------|
| 200 | 请求成功 |
| 400 | 参数错误 |
| 401 | Token无效或已过期 |
| 403 | 账号已禁用 |
| 404 | 资源不存在（用户不存在或接口不存在） |
| 500 | 系统异常 |

---

## 使用场景

### 场景1: 其他微服务获取当前用户信息

```java
// 在其他微服务中通过Feign调用
@FeignClient(name = "crm-operation-system", path = "/v1/internal")
public interface SystemClient {
    
    @GetMapping("/current-user")
    Result<UserVO> getCurrentUser(@RequestHeader("Authorization") String token);
}

// 使用示例
public void someBusinessMethod(String token) {
    Result<UserVO> result = systemClient.getCurrentUser("Bearer " + token);
    UserVO user = result.getData();
    // 使用用户信息...
}
```

### 场景2: 网关验证Token有效性

```java
// 在网关中验证Token
@Component
public class AuthFilter implements GlobalFilter {
    
    @Autowired
    private SystemClient systemClient;
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = exchange.getRequest().getHeaders().getFirst("Authorization");
        
        if (token != null && token.startsWith("Bearer ")) {
            String jwt = token.substring(7);
            
            // 调用内部接口验证Token
            TokenValidationDTO dto = new TokenValidationDTO();
            dto.setToken(jwt);
            
            Result<TokenValidationVO> result = systemClient.validateToken(dto);
            TokenValidationVO validation = result.getData();
            
            if (validation.getValid()) {
                // Token有效，添加用户信息到请求头
                ServerHttpRequest request = exchange.getRequest().mutate()
                    .header("X-User-Id", String.valueOf(validation.getUserId()))
                    .header("X-Username", validation.getUsername())
                    .build();
                
                return chain.filter(exchange.mutate().request(request).build());
            } else {
                // Token无效，返回401
                exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
                return exchange.getResponse().setComplete();
            }
        }
        
        return chain.filter(exchange);
    }
}
```

---

## 注意事项

1. **白名单配置**: 这些内部接口已添加到JWT拦截器白名单中，不需要经过身份认证拦截器

2. **Token格式**: 
   - `getCurrentUser` 接口从Header的Authorization字段获取Token（格式：`Bearer xxx`）
   - `validateToken` 接口从请求Body获取Token（纯Token字符串，不包含Bearer前缀）

3. **安全性**: 
   - 内部接口应该通过网关或IP白名单进行访问控制
   - 建议在生产环境中限制只能从内网访问

4. **性能**: 
   - `validateToken` 接口设计为高性能接口，适合高频调用
   - 平均响应时间 < 100ms

5. **错误处理**: 
   - `getCurrentUser` 接口遇到错误会抛出BusinessException
   - `validateToken` 接口始终返回200状态码，通过`valid`字段判断Token是否有效

---

## 测试用例

### 测试用例1: 验证有效Token

```bash
# 1. 登录获取Token
curl -X POST http://localhost:18101/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin123"}'

# Response:
# {
#   "code": 200,
#   "data": {
#     "token": "eyJhbGc..."
#   }
# }

# 2. 验证Token
curl -X POST http://localhost:18101/v1/internal/validate-token \
  -H "Content-Type: application/json" \
  -d '{"token":"eyJhbGc..."}'

# Response:
# {
#   "code": 200,
#   "data": {
#     "valid": true,
#     "userId": 1,
#     "username": "admin",
#     "realName": "系统管理员"
#   }
# }
```

### 测试用例2: 获取当前用户信息

```bash
curl -X GET http://localhost:18101/v1/internal/current-user \
  -H "Authorization: Bearer eyJhbGc..."

# Response:
# {
#   "code": 200,
#   "data": {
#     "id": 1,
#     "username": "admin",
#     "realName": "系统管理员",
#     ...
#   }
# }
```

### 测试用例3: 验证无效Token

```bash
curl -X POST http://localhost:18101/v1/internal/validate-token \
  -H "Content-Type: application/json" \
  -d '{"token":"invalid-token"}'

# Response:
# {
#   "code": 200,
#   "data": {
#     "valid": false,
#     "reason": "Token无效或已过期"
#   }
# }
```

---

## 更新日志

### v1.0.0 (2025-10-15)
- 初始版本发布
- 实现获取当前用户信息接口
- 实现Token验证接口
- 添加白名单配置

