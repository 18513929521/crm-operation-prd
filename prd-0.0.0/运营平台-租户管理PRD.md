# 运营平台-租户管理PRD

## 1. 文档信息

| 项目 | 内容 |
|------|------|
| 文档名称 | 运营平台-租户管理PRD |
| 文档版本 | 1.0 |
| 创建日期 | 2025-10-14 |
| 作者 | 技术团队 |
| 更新记录 | 初始版本 |

## 2. 模块概述

### 2.1 模块定位
租户管理模块是运营平台的核心功能模块之一，负责管理所有租户的生命周期，包括租户创建、信息维护、状态管理以及授权控制。

### 2.2 核心价值
1. 统一管理所有租户信息
2. 控制租户访问权限
3. 与NovoGrowth平台进行数据同步
4. 提供租户操作审计功能

## 3. 技术实现方案

### 3.1 前端技术栈

| 技术 | 说明 |
|------|------|
| Vue3 | 前端框架，提供响应式数据绑定和组件化开发 |
| Element Plus | UI组件库，提供丰富的界面组件 |
| Vite | 构建工具，提供快速的开发体验 |
| qiankun | 微前端框架，作为租户管理子应用集成到主应用 |
| vite-plugin-qiankun | Vite插件，支持qiankun微前端 |
| Pinia | 状态管理库，管理租户相关数据状态 |
| Axios | HTTP客户端，用于与后端API通信 |
| Less | CSS预处理器，提供更强大的样式编写能力 |

### 3.2 微前端集成方案

#### 3.2.1 应用注册
``javascript
// micro-app.js
const apps = [
  {
    name: 'tenant-management',
    entry: '//localhost:8083',
    container: '#subapp-container',
    activeRule: '/tenant',
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
    console.log('[tenant-management] bootstrap');
  },
  mount(props) {
    console.log('[tenant-management] mount', props);
    render(props);
  },
  update(props) {
    console.log('[tenant-management] update', props);
  },
  unmount() {
    console.log('[tenant-management] unmount');
    app.unmount();
    app = null;
  }
});
```

#### 3.2.3 子应用路由配置
``javascript
// router/index.js (子应用路由配置)
import { createRouter, createWebHistory, createWebHashHistory } from 'vue-router';
import { qiankunWindow } from 'vite-plugin-qiankun/dist/helper';

// 根据是否在qiankun环境中选择路由模式
const history = qiankunWindow.__POWERED_BY_QIANKUN__ 
  ? createWebHistory('/tenant/') 
  : createWebHistory();

const routes = [
  {
    path: '/',
    redirect: '/list'
  },
  {
    path: '/list',
    name: 'TenantList',
    component: () => import('@/views/tenant/List.vue'),
    meta: { title: '租户列表' }
  },
  {
    path: '/create',
    name: 'TenantCreate',
    component: () => import('@/views/tenant/Create.vue'),
    meta: { title: '创建租户' }
  },
  {
    path: '/detail/:id',
    name: 'TenantDetail',
    component: () => import('@/views/tenant/Detail.vue'),
    meta: { title: '租户详情' }
  },
  {
    path: '/authorize/:id',
    name: 'TenantAuthorize',
    component: () => import('@/views/tenant/Authorize.vue'),
    meta: { title: '租户授权' }
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
  name: 'tenant-management',
  entry: '//localhost:8083',
  container: '#subapp-container',
  activeRule: '/tenant',
  props: {
    routerBase: '/tenant',
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
  next();
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
  base: '/tenant',
  server: {
    port: 8083,
    cors: true,
    proxy: {
      '/api/tenant': {
        target: 'http://localhost:3003',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api\/tenant/, '')
      }
    }
  },
  plugins: [
    vue(),
    qiankun('tenant-management', {
      useDevMode: true
    })
  ]
});
```

#### 3.3.2 生产环境Nginx配置
``nginx
server {
    listen 80;
    server_name tenant.operation.novogrowth.com;
    
    location / {
        root /var/www/html/tenant;
        try_files $uri $uri/ /tenant/index.html;
    }
    
    location /api/tenant {
        proxy_pass http://tenant-service:3003;
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
<link rel="stylesheet" href="https://cdn.novogrowth.com/tenant/css/index.css">
<script src="https://cdn.novogrowth.com/tenant/js/index.js"></script>
```

#### 3.4.2 资源预加载
```
<!-- 关键资源预加载 -->
<link rel="preload" href="/tenant/assets/tenant-list.js" as="script">
<link rel="prefetch" href="/tenant/assets/tenant-detail.js" as="script">
```

## 4. 功能详细设计

### 4.1 租户列表功能

#### 4.1.1 功能描述
展示所有租户的基本信息，支持搜索、筛选和分页功能。

#### 4.1.2 界面原型
```
┌─────────────────────────────────────────────────────────────┐
│  租户管理                                                    │
├─────────────────────────────────────────────────────────────┤
│  [搜索框_______]  [状态筛选▼]  [导出]  [+ 创建租户]         │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ 企业名称  │ Tenant ID │ 授权状态│ 创建时间  │ 操作  │  │
│  ├──────────────────────────────────────────────────────┤  │
│  │ 测试企业A │T20251001  │ 已授权  │2025-10-01 │详情   │  │
│  │ 测试企业B │T20251002  │ 待授权  │2025-10-05 │详情   │  │
│  │ 测试企业C │T20251003  │ 已停用  │2025-10-08 │详情   │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  [1] [2] [3] ... [下一页] [跳转到__]                        │
└─────────────────────────────────────────────────────────────┘
```

#### 4.1.3 功能实现要点
1. 支持按企业名称、Tenant ID搜索
2. 支持按授权状态筛选（全部/待授权/已授权/已停用）
3. 表格分页展示，默认每页20条记录
4. 支持导出功能（Excel格式）
5. 点击"详情"跳转到租户详情页面

#### 4.1.4 API接口设计
```
GET /api/tenant/list
参数：
- page: 页码
- size: 每页条数
- keyword: 搜索关键字
- status: 状态筛选

响应：
{
  "code": 0,
  "data": {
    "list": [
      {
        "id": "T20251001",
        "companyName": "测试企业A",
        "tenantId": "T20251001",
        "authStatus": "AUTHORIZED",
        "createTime": "2025-10-01 10:00:00"
      }
    ],
    "total": 100,
    "page": 1,
    "size": 20
  }
}
```

### 4.2 租户创建功能

#### 4.2.1 功能描述
运营人员可以创建新的租户账号，系统会自动生成Tenant ID并调用NovoGrowth API创建主账号。

#### 4.2.2 界面原型
```
┌─────────────────────────────────────┐
│  创建租户                            │
├─────────────────────────────────────┤
│  企业名称：                          │
│  [________________]                  │
│                                      │
│  联系人姓名：                        │
│  [________________]                  │
│                                      │
│  邮箱：                              │
│  [________________]                  │
│                                      │
│  手机号：                            │
│  [________________]                  │
│                                      │
│  行业：                              │
│  [________________]                  │
│                                      │
│  备注：                              │
│  [__________________________]        │
│                                      │
│  [ 取消 ]  [ 保存 ]                 │
└─────────────────────────────────────┘
```

#### 4.2.3 功能实现要点
1. 表单验证：必填字段检查、邮箱格式验证、手机号格式验证
2. 邮箱唯一性校验
3. 自动生成Tenant ID（T+时间戳+随机数）
4. 调用NovoGrowth API创建主账号
5. 创建成功后发送通知邮件

#### 4.2.4 API接口设计
```
POST /api/tenant/create
参数：
{
  "companyName": "测试企业A",
  "contactName": "张三",
  "email": "admin@test.com",
  "phone": "13800138000",
  "industry": "互联网",
  "remark": "备注信息"
}

响应：
{
  "code": 0,
  "data": {
    "id": "T20251001",
    "tenantId": "T20251001",
    "status": "PENDING_AUTH"
  }
}
```

### 4.3 租户详情功能

#### 4.3.1 功能描述
展示租户的详细信息，包括基本信息、授权状态、NovoGrowth账号信息和操作记录。

#### 4.3.2 界面原型
```
┌─────────────────────────────────────────────────┐
│  租户详情 - 测试企业A                            │
├─────────────────────────────────────────────────┤
│                                                  │
│  基本信息                                        │
│  ├─ 企业名称：测试企业A                         │
│  ├─ Tenant ID：T202510100001                   │
│  ├─ 联系人：张三                                │
│  ├─ 邮箱：admin@testa.com                       │
│  ├─ 手机：13800138000                           │
│  ├─ 行业：互联网                                │
│  ├─ 创建时间：2025-10-01                        │
│  └─ 授权状态：已授权                            │
│                                                  │
│  授权状态：已授权测试版                        │
│                                                  │
│  授权模块（测试版）                              │
│  ✅ 业务需求分析                                │
│  ✅ 线索管理（完整版）                         │
│  ✅ 销售管理                                     │
│  ✅ 系统管理                                    │
│                                                  │
│  NovoGrowth账号                                 │
│  ├─ 账号ID：A123456                            │
│  ├─ 登录地址：https://app.novogrowth.com       │
│  └─ 状态：正常                                  │
│                                                  │
│  操作记录                                        │
│  └─ [时间线展示状态变更]                        │
│     2025-10-02  已授权测试版                    │
│     2025-10-01  创建租户账号                    │
│                                                  │
│  操作                                            │
│  [ 停用 ]  [ 删除 ]                             │
└─────────────────────────────────────────────────┘
```

#### 4.3.3 功能实现要点
1. 展示租户完整信息
2. 显示授权状态和模块详情
3. 展示NovoGrowth账号信息
4. 以时间线形式展示操作记录
5. 提供停用/删除操作按钮

#### 4.3.4 API接口设计
```
GET /api/tenant/detail/{tenantId}
响应：
{
  "code": 0,
  "data": {
    "basicInfo": {
      "companyName": "测试企业A",
      "tenantId": "T20251001",
      "contactName": "张三",
      "email": "admin@testa.com",
      "phone": "13800138000",
      "industry": "互联网",
      "createTime": "2025-10-01 10:00:00"
    },
    "authInfo": {
      "status": "AUTHORIZED",
      "modules": ["业务需求分析", "线索管理", "销售管理", "系统管理"]
    },
    "novogrowthInfo": {
      "accountId": "A123456",
      "loginUrl": "https://app.novogrowth.com",
      "status": "ACTIVE"
    },
    "operationLogs": [
      {
        "time": "2025-10-02 10:00:00",
        "action": "已授权测试版"
      },
      {
        "time": "2025-10-01 10:00:00",
        "action": "创建租户账号"
      }
    ]
  }
}
```

### 4.4 租户授权功能

#### 4.4.1 功能描述
为租户授权测试版配置，授权信息将同步到NovoGrowth平台。

#### 4.4.2 界面原型
```
┌─────────────────────────────────────┐
│  为租户授权（测试版）                │
├─────────────────────────────────────┤
│  租户信息：                          │
│  企业名称：测试企业B                 │
│  Tenant ID：T202510100002           │
│                                      │
│  ┌────────────────────────────────┐ │
│  │ 测试版配置：                    │ │
│  │ ✅ 业务需求分析                 │ │
│  │ ✅ 线索管理（完整版）          │ │
│  │ ✅ 销售管理                      │ │
│  │ ✅ 系统管理                     │ │
│  │                                 │ │
│  │ 配额：                          │ │
│  │ - 用户数：10                    │ │
│  │ - Leads数：5,000               │ │
│  │ - 客户数：1,000                │ │
│  │ - 存储空间：10GB                │ │
│  └────────────────────────────────┘ │
│                                      │
│  [ 取消 ]  [ 确认授权 ]             │
└─────────────────────────────────────┘
```

#### 4.4.3 功能实现要点
1. 展示租户基本信息
2. 显示测试版配置详情
3. 确认授权后构建授权配置
4. 推送配置到NovoGrowth平台
5. 更新租户授权状态
6. 发送通知邮件给租户

#### 4.4.4 API接口设计
```
POST /api/tenant/authorize
参数：
{
  "tenantId": "T20251002"
}

响应：
{
  "code": 0,
  "data": {
    "success": true,
    "message": "授权成功"
  }
}
```

## 5. 非功能性需求

### 5.1 性能要求
- 租户列表页面加载时间 < 1.5秒
- 租户详情页面加载时间 < 1秒
- 租户创建响应时间 < 500ms
- 租户授权同步到NovoGrowth < 1分钟

### 5.2 安全要求
- 租户信息传输加密
- 敏感操作需要权限验证
- 操作日志完整记录
- 防止SQL注入和XSS攻击

### 5.3 可用性要求
- 租户管理模块可用性 ≥ 99.9%
- 故障恢复时间 < 30分钟
- 数据备份每日执行

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
| 需求分析 | 2025-10-15 | 租户管理PRD完成 |
| 技术设计 | 2025-10-20 | 技术方案确定 |
| 前端开发 | 2025-11-10 | 前端功能开发完成 |
| 后端开发 | 2025-11-15 | 后端接口开发完成 |
| 联调测试 | 2025-11-20 | 前后端联调完成 |
| 系统测试 | 2025-11-25 | 系统测试通过 |
| 上线部署 | 2025-12-01 | 正式上线运行 |