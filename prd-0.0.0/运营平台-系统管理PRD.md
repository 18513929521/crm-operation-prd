# 运营平台-系统管理PRD

## 1. 文档信息

| 项目 | 内容 |
|------|------|
| 文档名称 | 运营平台-系统管理PRD |
| 文档版本 | 1.0 |
| 创建日期 | 2025-10-14 |
| 作者 | 技术团队 |
| 更新记录 | 初始版本 |

## 2. 模块概述

### 2.1 模块定位
系统管理模块是运营平台的基础功能模块，负责平台自身的用户管理、权限控制、登录认证、会话管理和系统配置等核心功能。

### 2.2 核心价值
1. 提供安全的用户认证和授权机制
2. 管理运营平台的用户和权限
3. 记录和审计所有操作日志
4. 配置和管理系统运行参数

## 3. 技术实现方案

### 3.1 前端技术栈

| 技术 | 说明 |
|------|------|
| Vue3 | 前端框架，提供响应式数据绑定和组件化开发 |
| Element Plus | UI组件库，提供丰富的界面组件 |
| Vite | 构建工具，提供快速的开发体验 |
| qiankun | 微前端框架，作为系统管理子应用集成到主应用 |
| vite-plugin-qiankun | Vite插件，支持qiankun微前端 |
| Pinia | 状态管理库，管理系统相关数据状态 |
| Axios | HTTP客户端，用于与后端API通信 |
| Less | CSS预处理器，提供更强大的样式编写能力 |

### 3.2 微前端集成方案

#### 3.2.1 应用注册
``javascript
// micro-app.js
const apps = [
  {
    name: 'system-management',
    entry: '//localhost:8081',
    container: '#subapp-container',
    activeRule: '/system',
  }
];
```

#### 3.2.2 生命周期管理
``javascript
// main.js (子应用入口)
import { renderWithQiankun, qiankunWindow } from 'vite-plugin-qiankun/dist/helper';
import { createApp } from 'vue';
import App from './App.vue';
import router from './router';
import store from './store';

let app = null;

function render(props = {}) {
  const { container } = props;
  app = createApp(App);
  app.use(store);
  app.use(router);
  app.mount(container ? container.querySelector('#app') : '#app');
}

if (!qiankunWindow.__POWERED_BY_QIANKUN__) {
  // 独立运行
  render();
}

// qiankun生命周期钩子
renderWithQiankun({
  bootstrap() {
    console.log('[system-management] bootstrap');
  },
  mount(props) {
    console.log('[system-management] mount', props);
    render(props);
  },
  update(props) {
    console.log('[system-management] update', props);
  },
  unmount() {
    console.log('[system-management] unmount');
    app.unmount();
    app = null;
  }
});
```

#### 3.2.3 子应用路由配置
``javascript
// router/index.js (子应用路由配置)
import { createRouter, createWebHistory } from 'vue-router';
import { qiankunWindow } from 'vite-plugin-qiankun/dist/helper';

// 根据是否在qiankun环境中选择路由模式
const history = qiankunWindow.__POWERED_BY_QIANKUN__ 
  ? createWebHistory('/system/') 
  : createWebHistory();

const routes = [
  {
    path: '/',
    redirect: '/user'
  },
  {
    path: '/user',
    name: 'UserManagement',
    component: () => import('@/views/user/List.vue'),
    meta: { title: '用户管理' }
  },
  {
    path: '/user/create',
    name: 'UserCreate',
    component: () => import('@/views/user/Create.vue'),
    meta: { title: '创建用户' }
  },
  {
    path: '/user/edit/:id',
    name: 'UserEdit',
    component: () => import('@/views/user/Edit.vue'),
    meta: { title: '编辑用户' }
  },
  {
    path: '/auth',
    name: 'AuthManagement',
    component: () => import('@/views/auth/Login.vue'),
    meta: { title: '登录认证' }
  },
  {
    path: '/session',
    name: 'SessionManagement',
    component: () => import('@/views/session/List.vue'),
    meta: { title: '会话管理' }
  },
  {
    path: '/log',
    name: 'LogAudit',
    component: () => import('@/views/log/List.vue'),
    meta: { title: '日志审计' }
  },
  {
    path: '/config',
    name: 'SystemConfig',
    component: () => import('@/views/config/Index.vue'),
    meta: { title: '系统配置' }
  }
];

const router = createRouter({
  history,
  routes
});

export default router;
```

#### 3.2.4 主应用与子应用通信
``javascript
// 主应用传递给子应用的props
{
  name: 'system-management',
  entry: '//localhost:8081',
  container: '#subapp-container',
  activeRule: '/system',
  props: {
    routerBase: '/system',
    // 全局状态
    globalState: {
      user: null,
      permissions: []
    },
    // 全局状态变更回调
    onGlobalStateChange: (state, prevState) => {
      console.log('global state changed', state, prevState);
    },
    // 设置全局状态
    setGlobalState: (state) => {
      console.log('set global state', state);
    }
  }
}

// 子应用中获取主应用传递的props
// 在路由守卫中使用
router.beforeEach((to, from, next) => {
  // 可以在这里获取主应用传递的数据
  console.log('Router guard - global state:', window.globalState);
  
  // 权限校验
  const requiredPermissions = to.meta.permissions;
  if (requiredPermissions && !hasPermission(requiredPermissions)) {
    next('/auth/login');
  } else {
    next();
  }
});
```

### 3.3 跨域解决方案

#### 3.3.1 开发环境代理配置
``javascript
// vite.config.js (子应用配置)
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import qiankun from 'vite-plugin-qiankun';

export default defineConfig({
  base: '/system',
  server: {
    port: 8081,
    cors: true,
    proxy: {
      '/api/system': {
        target: 'http://localhost:3001',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api\/system/, '')
      }
    }
  },
  plugins: [
    vue(),
    qiankun('system-management', {
      useDevMode: true
    })
  ]
});
```

#### 3.3.2 生产环境Nginx配置
``nginx
server {
    listen 80;
    server_name system.operation.novogrowth.com;
    
    location / {
        root /var/www/html/system;
        try_files $uri $uri/ /system/index.html;
    }
    
    location /api/system {
        proxy_pass http://system-service:3001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 3.4 海外访问优化

#### 3.4.1 CDN资源配置
```
<!-- 静态资源使用CDN -->
<link rel="stylesheet" href="https://cdn.novogrowth.com/system/css/index.css">
<script src="https://cdn.novogrowth.com/system/js/index.js"></script>
```

#### 3.4.2 资源预加载
```
<!-- 关键资源预加载 -->
<link rel="preload" href="/system/assets/user-list.js" as="script">
<link rel="prefetch" href="/system/assets/permission-config.js" as="script">
```

## 4. 功能详细设计

### 4.1 用户管理功能

#### 4.1.1 功能描述
管理系统运营人员账号，包括用户创建、编辑、状态管理等功能。

#### 4.1.2 界面原型
```
┌─────────────────────────────────────────────────────────────┐
│  运营人员管理                                                │
├─────────────────────────────────────────────────────────────┤
│  [添加用户]  [批量导入]  [导出]                             │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ 姓名  │ 账号         │ 角色      │ 状态  │ 最后登录 │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │ 李四  │admin          │超级管理员 │启用   │2小时前   │  │
│  │ 王五  │ops            │运营人员   │启用   │1天前     │  │
│  │ 张三  │test           │运营人员   │禁用   │7天前     │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

#### 4.1.3 功能实现要点
1. 用户列表展示（支持搜索和筛选）
2. 用户创建/编辑/禁用
3. 密码重置功能
4. 用户角色分配
5. 支持批量导入和导出

#### 4.1.4 API接口设计例子
```
GET /api/system/user/list
参数：
- page: 页码
- size: 每页条数
- keyword: 搜索关键字

响应：
{
  "code": 0,
  "data": {
    "list": [
      {
        "id": 1,
        "name": "李四",
        "email": "admin        ",
        "role": "SUPER_ADMIN",
        "status": "ENABLED",
        "lastLoginTime": "2025-10-14 10:00:00"
      }
    ],
    "total": 100,
    "page": 1,
    "size": 20
  }
}
```

### 4.2 登录认证功能

#### 4.2.1 功能描述
提供运营平台的登录认证功能，包括账号密码登录、Token机制和会话管理。

#### 4.2.2 界面原型
```
┌─────────────────────────────────────┐
│  租户运营平台                        │
│                                      │
│  账号：                              │
│  [________________]                  │
│                                      │
│  密码：                              │
│  [________________]                  │
│                                      │
│  ☑ 记住登录状态（7天）               │
│                                      │
│  [ 登录 ]                            │
│                                      │
│  忘记密码？                          │
└─────────────────────────────────────┘
```

#### 4.2.3 功能实现要点
1. 账号密码登录验证
2. Token机制实现会话管理
3. 记住登录状态（7天）
4. 密码错误锁定机制（5次错误锁定30分钟）
5. 登录日志记录

#### 4.2.4 API接口设计例子
```
POST /api/system/auth/login
参数：
{
  "email": "admin        ",
  "password": "password123",
  "rememberMe": true
}

响应：
{
  "code": 0,
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "userInfo": {
      "id": 1,
      "name": "李四",
      "email": "admin        ",
      "role": "SUPER_ADMIN"
    }
  }
}
```

### 4.3 权限控制功能

#### 4.3.1 功能描述
实现基于角色的访问控制（RBAC），包括菜单权限动态渲染、操作按钮权限控制和API接口权限校验。

#### 4.3.2 功能实现要点
1. 基于角色的访问控制（RBAC）
2. 菜单权限动态渲染
3. 操作按钮权限控制
4. API接口权限校验

#### 4.3.3 API接口设计例子
```
GET /api/system/auth/permissions
响应：
{
  "code": 0,
  "data": {
    "menus": [
      {
        "id": 1,
        "name": "系统管理",
        "path": "/system",
        "children": [
          {
            "id": 2,
            "name": "用户管理",
            "path": "/system/user"
          }
        ]
      }
    ],
    "permissions": [
      "system:user:create",
      "system:user:edit",
      "system:user:delete"
    ]
  }
}
```

### 4.4 会话管理功能

#### 4.4.1 功能描述
管理用户的登录会话，支持查看当前登录设备和踢出指定设备。

#### 4.4.2 界面原型
```
┌─────────────────────────────────────────────────────────────┐
│  会话管理                                                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ 设备信息      │ 登录时间    │ 最后活动  │ 状态 │ 操作 │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │ Chrome/Windows│ 2小时前     │ 刚刚      │活跃 │踢出 │  │
│  │ Safari/Mac    │ 1天前       │ 30分钟前  │活跃 │踢出 │  │
│  │ Firefox/Linux │ 3天前       │ 2天前     │过期 │踢出 │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  [ 踢出所有设备 ]                                            │
└─────────────────────────────────────────────────────────────┘
```

#### 4.4.3 功能实现要点
1. 查看当前登录设备
2. 支持踢出指定设备
3. 支持强制退出所有设备
4. 会话过期自动清理

#### 4.4.4 API接口设计例子
```
GET /api/system/session/list
响应：
{
  "code": 0,
  "data": [
    {
      "id": 1,
      "deviceInfo": "Chrome/Windows",
      "loginTime": "2025-10-14 10:00:00",
      "lastActiveTime": "2025-10-14 12:00:00",
      "status": "ACTIVE"
    }
  ]
}
```

### 4.5 日志审计功能

#### 4.5.1 功能描述
记录和审计所有用户操作，包括登录日志、操作日志等。

#### 4.5.2 界面原型
```
┌─────────────────────────────────────────────────────────────┐
│  操作日志                                                    │
├─────────────────────────────────────────────────────────────┤
│  [时间范围] [操作人] [操作类型] [搜索]                      │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ 时间      │ 操作人 │ 操作类型 │ 操作内容 │ IP地址 │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │ 10:30:15  │ 李四   │ 用户管理 │ 创建用户王五 │ 192.168.1.100 │  │
│  │ 10:25:30  │ 李四   │ 租户管理 │ 邀请租户测试企业A │ 192.168.1.100 │  │
│  │ 10:20:45  │ 王五   │ 应用管理 │ 编辑菜单线索管理 │ 192.168.1.101 │  │
│  │ 09:15:20  │ 李四   │ 系统登录 │ 登录成功 │ 192.168.1.100 │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

#### 4.5.3 功能实现要点
1. 操作日志记录
2. 登录日志追踪
3. 日志搜索和导出
4. 敏感操作日志不可删除

#### 4.5.4 API接口设计例子
```
GET /api/system/log/list
参数：
- page: 页码
- size: 每页条数
- startTime: 开始时间
- endTime: 结束时间
- operator: 操作人
- type: 操作类型

响应：
{
  "code": 0,
  "data": {
    "list": [
      {
        "id": 1,
        "time": "2025-10-14 10:30:15",
        "operator": "李四",
        "type": "用户管理",
        "content": "创建用户王五",
        "ip": "192.168.1.100"
      }
    ],
    "total": 100,
    "page": 1,
    "size": 20
  }
}
```

### 4.6 系统配置功能

#### 4.6.1 功能描述
管理系统运行所需的各种配置参数，包括基础配置、安全配置、集成配置和邮件配置等。

#### 4.6.2 界面原型
```
┌─────────────────────────────────────────────────────────────┐
│  系统配置                                                    │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  基础配置                                                    │
│  ├─ 系统名称：[租户运营平台________________]                │
│  ├─ 系统Logo：[上传Logo]                                    │
│  ├─ 默认语言：[中文▼]                                       │
│  └─ 时区设置：[Asia/Shanghai▼]                             │
│                                                              │
│  安全配置                                                    │
│  ├─ 密码策略：                                              │
│  │  ├─ 最小长度：[8] 位                                      │
│  │  ├─ 复杂度要求：☑ 包含大小写字母、数字、特殊字符         │
│  │  └─ 有效期：[90] 天                                       │
│  ├─ 登录策略：                                              │
│  │  ├─ 最大失败次数：[5] 次                                  │
│  │  ├─ 锁定时间：[30] 分钟                                   │
│  │  └─ 会话超时：[24] 小时                                  │
│  └─ 并发会话限制：[5] 个                                     │
│                                                              │
│  集成配置                                                    │
│  ├─ NovoGrowth API地址：[https://api.novogrowth.com]        │
│  ├─ API超时时间：[30] 秒                                     │
│  └─ 重试次数：[3] 次                                         │
│                                                              │
│  邮件配置                                                    │
│  ├─ SMTP服务器：[smtp.company.com]                          │
│  ├─ 端口：[587]                                             │
│  ├─ 用户名：[noreply@company.com]                           │
│  └─ 密码：[________________]                                │
│                                                              │
│  [ 保存配置 ]                                                │
└─────────────────────────────────────────────────────────────┘
```

#### 4.6.3 功能实现要点
1. 基础配置管理
2. 安全策略配置
3. 集成配置管理
4. 邮件配置管理
5. 配置修改需要超级管理员权限

#### 4.6.4 API接口设计例子
```
GET /api/system/config
响应：
{
  "code": 0,
  "data": {
    "basic": {
      "systemName": "租户运营平台",
      "systemLogo": "/logo.png",
      "defaultLanguage": "zh-CN",
      "timezone": "Asia/Shanghai"
    },
    "security": {
      "passwordMinLength": 8,
      "passwordComplexity": true,
      "passwordValidity": 90,
      "maxLoginFailures": 5,
      "lockoutDuration": 30,
      "sessionTimeout": 24,
      "maxConcurrentSessions": 5
    },
    "integration": {
      "novogrowthApiUrl": "https://api.novogrowth.com",
      "apiTimeout": 30,
      "retryCount": 3
    },
    "email": {
      "smtpServer": "smtp.company.com",
      "port": 587,
      "username": "noreply@company.com"
    }
  }
}

POST /api/system/config/save
参数：
{
  "basic": {...},
  "security": {...},
  "integration": {...},
  "email": {...}
}
```

## 5. 非功能性需求

### 5.1 性能要求
- 用户列表页面加载时间 < 1.5秒
- 登录响应时间 < 500ms
- 权限校验响应时间 < 200ms
- 日志查询响应时间 < 1秒

### 5.2 安全要求
- 密码加密存储（BCrypt）
- Token过期机制（24小时）
- 敏感操作二次确认
- 防止暴力破解（5次错误锁定30分钟）
- 数据传输加密（HTTPS）

### 5.3 可用性要求
- 系统管理模块可用性 ≥ 99.9%
- 故障恢复时间 < 30分钟
- 数据备份每日执行
- 日志保存期限1年

## 6. 部署方案

### 6.1 开发环境
- Node.js >= 16.x
- npm >= 8.x
- Vite >= 4.x

### 6.2 生产环境
- Docker容器化部署
- Nginx反向代理
- Jenkins自动化部署

## 7. 项目里程碑

| 阶段 | 时间 | 交付内容 |
|------|------|----------|
| 需求分析 | 2025-10-15 | 系统管理PRD完成 |
| 技术设计 | 2025-10-20 | 技术方案确定 |
| 前端开发 | 2025-11-10 | 前端功能开发完成 |
| 后端开发 | 2025-11-15 | 后端接口开发完成 |
| 联调测试 | 2025-11-20 | 前后端联调完成 |
| 系统测试 | 2025-11-25 | 系统测试通过 |
| 上线部署 | 2025-12-01 | 正式上线运行 |