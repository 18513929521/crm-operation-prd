# 运营平台-应用管理PRD

## 1. 文档信息

| 项目 | 内容 |
|------|------|
| 文档名称 | 运营平台-应用管理PRD |
| 文档版本 | 1.0 |
| 创建日期 | 2025-10-14 |
| 作者 | 技术团队 |
| 更新记录 | 初始版本 |

## 2. 模块概述

### 2.1 模块定位
应用管理模块是运营平台的核心功能模块，负责管理NovoGrowth平台的应用配置，包括菜单管理、权限管理和测试版配置等功能。

### 2.2 核心价值
1. 统一管理NovoGrowth平台的功能模块
2. 配置和维护菜单结构
3. 管理操作权限
4. 定义和管理测试版配置

## 3. 技术实现方案

### 3.1 前端技术栈

| 技术 | 说明 |
|------|------|
| Vue3 | 前端框架，提供响应式数据绑定和组件化开发 |
| Element Plus | UI组件库，提供丰富的界面组件 |
| Vite | 构建工具，提供快速的开发体验 |
| qiankun | 微前端框架，作为应用管理子应用集成到主应用 |
| vite-plugin-qiankun | Vite插件，支持qiankun微前端 |
| Pinia | 状态管理库，管理应用相关数据状态 |
| Axios | HTTP客户端，用于与后端API通信 |
| Less | CSS预处理器，提供更强大的样式编写能力 |

### 3.2 微前端集成方案

#### 3.2.1 应用注册
```javascript
// micro-app.js
const apps = [
  {
    name: 'application-management',
    entry: '//localhost:8082',
    container: '#subapp-container',
    activeRule: '/app',
  }
];
```

#### 3.2.2 生命周期管理
```javascript
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
    console.log('[application-management] bootstrap');
  },
  mount(props) {
    console.log('[application-management] mount', props);
    render(props);
  },
  update(props) {
    console.log('[application-management] update', props);
  },
  unmount() {
    console.log('[application-management] unmount');
    app.unmount();
    app = null;
  }
});
```

#### 3.2.3 子应用路由配置
```javascript
// router/index.js (子应用路由配置)
import { createRouter, createWebHistory } from 'vue-router';
import { qiankunWindow } from 'vite-plugin-qiankun/dist/helper';

// 根据是否在qiankun环境中选择路由模式
const history = qiankunWindow.__POWERED_BY_QIANKUN__ 
  ? createWebHistory('/app/') 
  : createWebHistory();

const routes = [
  {
    path: '/',
    redirect: '/menu'
  },
  {
    path: '/menu',
    name: 'MenuManagement',
    component: () => import('@/views/menu/List.vue'),
    meta: { title: '菜单管理' }
  },
  {
    path: '/menu/create',
    name: 'MenuCreate',
    component: () => import('@/views/menu/Create.vue'),
    meta: { title: '创建菜单' }
  },
  {
    path: '/menu/edit/:id',
    name: 'MenuEdit',
    component: () => import('@/views/menu/Edit.vue'),
    meta: { title: '编辑菜单' }
  },
  {
    path: '/permission',
    name: 'PermissionManagement',
    component: () => import('@/views/permission/List.vue'),
    meta: { title: '权限管理' }
  },
  {
    path: '/permission/create',
    name: 'PermissionCreate',
    component: () => import('@/views/permission/Create.vue'),
    meta: { title: '创建权限' }
  },
  {
    path: '/permission/edit/:id',
    name: 'PermissionEdit',
    component: () => import('@/views/permission/Edit.vue'),
    meta: { title: '编辑权限' }
  },
  {
    path: '/config',
    name: 'TestVersionConfig',
    component: () => import('@/views/config/List.vue'),
    meta: { title: '测试版配置' }
  }
];

const router = createRouter({
  history,
  routes
});

export default router;
```

#### 3.2.4 主应用与子应用通信
```javascript
// 主应用传递给子应用的props
{
  name: 'application-management',
  entry: '//localhost:8082',
  container: '#subapp-container',
  activeRule: '/app',
  props: {
    routerBase: '/app',
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
```javascript
// vite.config.js (子应用配置)
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import qiankun from 'vite-plugin-qiankun';

export default defineConfig({
  base: '/app',
  server: {
    port: 8082,
    cors: true,
    proxy: {
      '/api/app': {
        target: 'http://localhost:3002',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api\/app/, '')
      }
    }
  },
  plugins: [
    vue(),
    qiankun('application-management', {
      useDevMode: true
    })
  ]
});
```

#### 3.3.2 生产环境Nginx配置
```nginx
server {
    listen 80;
    server_name app.operation.novogrowth.com;
    
    location / {
        root /var/www/html/app;
        try_files $uri $uri/ /app/index.html;
    }
    
    location /api/app {
        proxy_pass http://app-service:3002;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### 3.4 海外访问优化

#### 3.4.1 CDN资源配置
```html
<!-- 静态资源使用CDN -->
<link rel="stylesheet" href="https://cdn.novogrowth.com/app/css/index.css">
<script src="https://cdn.novogrowth.com/app/js/index.js"></script>
```

#### 3.4.2 资源预加载
```html
<!-- 关键资源预加载 -->
<link rel="preload" href="/app/assets/menu-list.js" as="script">
<link rel="prefetch" href="/app/assets/permission-config.js" as="script">
```

## 4. 功能详细设计

### 4.1 菜单管理功能

#### 4.1.1 功能描述
管理NovoGrowth平台的所有菜单项，包括菜单的层级结构、访问路径和状态。

#### 4.1.2 功能实现要点
1. 菜单树形结构展示
2. 菜单创建/编辑/删除
3. 菜单状态控制（启用/停用）
4. 菜单排序和层级管理
5. 菜单name全局唯一校验

#### 4.1.3 API接口设计例子
```
GET /api/app/menu/list
参数：
- parentId: 父级菜单ID（可选，用于获取子菜单）

响应：
{
  "code": 0,
  "data": [
    {
      "id": 1,
      "name": "业务需求分析",
      "type": "DIRECTORY",
      "icon": "home",
      "path": "/business",
      "status": "ENABLED",
      "sort": 1,
      "parentId": null,
      "children": [
        {
          "id": 2,
          "name": "自然语言转化",
          "type": "MENU",
          "icon": "nlp_transform",
          "path": "/business/nlp",
          "status": "ENABLED",
          "sort": 1,
          "parentId": 1
        }
      ]
    }
  ]
}

POST /api/app/menu/create
参数：
{
  "name": "新菜单",
  "type": "MENU",
  "icon": "menu",
  "path": "/new-menu",
  "status": "ENABLED",
  "sort": 1,
  "parentId": 1
}

响应：
{
  "code": 0,
  "data": {
    "id": 3,
    "name": "新菜单",
    "type": "MENU",
    "icon": "menu",
    "path": "/new-menu",
    "status": "ENABLED",
    "sort": 1,
    "parentId": 1
  }
}
```

### 4.2 权限管理功能

#### 4.2.1 功能描述
管理NovoGrowth平台的所有操作权限，权限与菜单关联。

#### 4.2.2 功能实现要点
1. 权限列表展示
2. 权限创建/编辑/删除
3. 权限与菜单关联
4. 权限状态控制
5. 权限ID全局唯一校验（格式：模块:功能:操作）

#### 4.2.3 API接口设计例子
```
GET /api/app/permission/list
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
        "name": "使用自然语言转化",
        "type": "QUERY",
        "permissionId": "business:nlp:use",
        "access": "ALL_AGENTS",
        "menuId": 2,
        "moduleName": "业务需求分析",
        "sort": 1,
        "status": "ENABLED"
      }
    ],
    "total": 100,
    "page": 1,
    "size": 20
  }
}

POST /api/app/permission/create
参数：
{
  "name": "新权限",
  "type": "QUERY",
  "permissionId": "module:feature:action",
  "access": "ALL_AGENTS",
  "menuId": 1,
  "sort": 1,
  "status": "ENABLED"
}

响应：
{
  "code": 0,
  "data": {
    "id": 2,
    "name": "新权限",
    "type": "QUERY",
    "permissionId": "module:feature:action",
    "access": "ALL_AGENTS",
    "menuId": 1,
    "moduleName": "业务需求分析",
    "sort": 1,
    "status": "ENABLED"
  }
}
```


## 5. 非功能性需求

### 5.1 性能要求
- 菜单列表页面加载时间 < 1.5秒
- 权限列表页面加载时间 < 1.5秒
- 配置详情页面加载时间 < 1秒
- 菜单创建/编辑响应时间 < 500ms
- 权限创建/编辑响应时间 < 500ms

### 5.2 安全要求
- 菜单名称和权限ID唯一性校验
- 敏感操作需要权限验证
- 操作日志完整记录
- 防止SQL注入和XSS攻击

### 5.3 可用性要求
- 应用管理模块可用性 ≥ 99.9%
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
| 需求分析 | 2025-10-15 | 应用管理PRD完成 |
| 技术设计 | 2025-10-20 | 技术方案确定 |
| 前端开发 | 2025-11-12 | 前端功能开发完成 |
| 后端开发 | 2025-11-17 | 后端接口开发完成 |
| 联调测试 | 2025-11-22 | 前后端联调完成 |
| 系统测试 | 2025-11-27 | 系统测试通过 |
| 上线部署 | 2025-12-01 | 正式上线运行 |