# 运营平台PRD

## 1. 文档信息

| 项目 | 内容 |
|------|------|
| 文档名称 | 运营平台PRD |
| 文档版本 | 1.0 |
| 创建日期 | 2025-10-14 |
| 作者 | 前端团队 |

## 2. 产品概述

### 2.1 产品愿景
打造一款以应用管理为核心的SaaS租户运营平台，支持NovoGrowth平台的应用配置、租户管理和授权控制，快速验证商业模式。

### 2.2 产品定位
运营平台是NovoGrowth生态系统的重要组成部分，负责管理租户生命周期、应用配置和权限控制。

## 3. 技术架构

### 3.1 前端技术栈

| 技术 | 说明 |
|------|------|
| Vue3 | 前端框架，提供响应式数据绑定和组件化开发 |
| Element Plus | UI组件库，提供丰富的界面组件 |
| Vite | 构建工具，提供快速的开发体验 |
| qiankun | 微前端框架，实现主子应用集成 |
| vite-plugin-qiankun | Vite插件，支持qiankun微前端 |
| Pinia | 状态管理库，替代Vuex的轻量级方案 |
| Axios | HTTP客户端，用于API请求 |
| Less | CSS预处理器，提供更强大的样式编写能力 |

### 3.2 微前端架构设计

#### 3.2.1 架构说明
运营平台采用微前端架构，主应用负责整体布局、导航和权限控制，各个功能模块作为独立的子应用进行开发和部署。

#### 3.2.2 应用划分
- 主应用：负责整体框架、登录认证、菜单导航
- 系统管理子应用：用户管理、权限管理、日志审计等
- 应用管理子应用：菜单管理、权限配置等
- 租户管理子应用：租户列表、租户详情、授权管理等

#### 3.2.3 主应用qiankun配置

##### 3.2.3.1 主应用初始化配置
```javascript
// main.js - 主应用入口文件
import { registerMicroApps, start, setDefaultMountApp } from 'qiankun';
import apps from './micro-apps';

// 注册子应用
registerMicroApps(apps, {
  beforeLoad: app => {
    console.log('[MainApp] before load %c%s', 'color: green;', app.name);
    return Promise.resolve();
  },
  beforeMount: app => {
    console.log('[MainApp] before mount %c%s', 'color: green;', app.name);
    return Promise.resolve();
  },
  afterUnmount: app => {
    console.log('[MainApp] after unmount %c%s', 'color: green;', app.name);
    return Promise.resolve();
  }
});

// 设置默认子应用
setDefaultMountApp('/system');

// 启动qiankun
start({
  prefetch: 'all', // 预加载所有子应用静态资源
  sandbox: {
    strictStyleIsolation: false, // 是否开启严格样式隔离
    experimentalStyleIsolation: true // 实验性样式隔离
  },
  singular: true // 是否为单实例场景
});
```

##### 3.2.3.2 子应用注册配置
```javascript
// micro-apps.js - 子应用配置文件
const apps = [
  {
    name: 'system-management', // 子应用名称
    entry: process.env.NODE_ENV === 'development' 
      ? '//localhost:8081' 
      : 'https://system.operation.novogrowth.com', // 子应用入口
    container: '#subapp-container', // 子应用挂载容器
    activeRule: '/system', // 子应用激活规则
    props: {
      // 传递给子应用的参数
      routerBase: '/system',
      globalState: {
        user: null,
        permissions: []
      }
    }
  },
  {
    name: 'application-management',
    entry: process.env.NODE_ENV === 'development' 
      ? '//localhost:8082' 
      : 'https://app.operation.novogrowth.com',
    container: '#subapp-container',
    activeRule: '/app',
    props: {
      routerBase: '/app'
    }
  },
  {
    name: 'tenant-management',
    entry: process.env.NODE_ENV === 'development' 
      ? '//localhost:8083' 
      : 'https://tenant.operation.novogrowth.com',
    container: '#subapp-container',
    activeRule: '/tenant',
    props: {
      routerBase: '/tenant'
    }
  }
];

export default apps;
```

##### 3.2.3.3 主应用路由配置
```javascript
// router/index.js - 主应用路由配置
import { createRouter, createWebHistory } from 'vue-router';
import Layout from '@/layout/index.vue';

const routes = [
  {
    path: '/',
    component: Layout,
    redirect: '/system',
    children: [
      {
        path: '/system',
        name: 'System',
        component: () => import('@/views/system/index.vue'),
        meta: { title: '系统管理' }
      },
      {
        path: '/app',
        name: 'Application',
        component: () => import('@/views/application/index.vue'),
        meta: { title: '应用管理' }
      },
      {
        path: '/tenant',
        name: 'Tenant',
        component: () => import('@/views/tenant/index.vue'),
        meta: { title: '租户管理' }
      }
    ]
  },
  {
    path: '/login',
    name: 'Login',
    component: () => import('@/views/login/index.vue'),
    meta: { title: '登录' }
  },
  // 重定向到子应用的路由配置
  {
    path: '/system/:catchAll(.*)',
    redirect: to => {
      return `/system`;
    }
  },
  {
    path: '/app/:catchAll(.*)',
    redirect: to => {
      return `/app`;
    }
  },
  {
    path: '/tenant/:catchAll(.*)',
    redirect: to => {
      return `/tenant`;
    }
  }
];

const router = createRouter({
  history: createWebHistory(),
  routes
});

export default router;
```

#### 3.2.4 跨域解决方案
1. 开发环境：通过配置Vite代理解决跨域问题
2. 生产环境：通过Nginx反向代理统一域名，避免跨域

##### 3.2.4.1 开发环境Vite代理配置
```javascript
// vite.config.js
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import qiankun from 'vite-plugin-qiankun';

export default defineConfig({
  server: {
    port: 8080,
    proxy: {
      // 系统管理API代理
      '/api/system': {
        target: 'http://localhost:3001',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api\/system/, '')
      },
      // 应用管理API代理
      '/api/app': {
        target: 'http://localhost:3002',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api\/app/, '')
      },
      // 租户管理API代理
      '/api/tenant': {
        target: 'http://localhost:3003',
        changeOrigin: true,
        rewrite: (path) => path.replace(/^\/api\/tenant/, '')
      }
    }
  },
  plugins: [
    vue(),
    qiankun('main-app', {
      useDevMode: true
    })
  ]
});
```

##### 3.2.4.2 生产环境Nginx配置
```nginx
# nginx.conf
server {
    listen 80;
    server_name operation.novogrowth.com;
    
    # 主应用静态资源
    location / {
        root /var/www/html/main;
        try_files $uri $uri/ /index.html;
    }
    
    # 系统管理子应用
    location /system {
        root /var/www/html/system;
        try_files $uri $uri/ /system/index.html;
    }
    
    # 应用管理子应用
    location /app {
        root /var/www/html/app;
        try_files $uri $uri/ /app/index.html;
    }
    
    # 租户管理子应用
    location /tenant {
        root /var/www/html/tenant;
        try_files $uri $uri/ /tenant/index.html;
    }
    
    # API代理配置
    location /api/system {
        proxy_pass http://system-service:3001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    location /api/app {
        proxy_pass http://app-service:3002;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
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

#### 3.2.5 海外访问CDN策略
1. 静态资源（JS/CSS/图片）部署到CDN
2. 根据用户地理位置选择最近的CDN节点
3. 关键资源使用preload/prefetch提升加载速度
4. 资源文件启用gzip压缩和缓存策略

### 3.3 自动化部署

#### 3.3.1 Jenkins流水线配置
```groovy
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'git@github.com:your-company/crm-operation-web.git'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
        
        stage('Deploy') {
            steps {
                sh 'scp -r dist/* user@server:/var/www/html'
            }
        }
    }
}
```

#### 3.3.2 部署策略
1. 使用Docker容器化部署
2. 通过Nginx进行负载均衡
3. 实现蓝绿部署，确保零停机更新

## 4. 功能模块设计

### 4.1 系统管理模块

#### 4.1.1 用户管理
- 用户列表展示（支持搜索和筛选）
- 用户创建/编辑/禁用
- 密码重置功能
- 用户角色分配

#### 4.1.2 登录认证
- 账号密码登录
- Token机制实现会话管理
- 记住登录状态（7天）
- 密码错误锁定机制（5次错误锁定30分钟）

#### 4.1.3 权限控制
- 基于角色的访问控制（RBAC）
- 菜单权限动态渲染
- 操作按钮权限控制
- API接口权限校验

#### 4.1.4 会话管理
- 查看当前登录设备
- 支持踢出指定设备
- 强制退出所有设备

#### 4.1.5 日志审计
- 操作日志记录
- 登录日志追踪
- 日志搜索和导出

#### 4.1.6 系统配置
- 基础配置管理
- 安全策略配置
- 集成配置管理
- 邮件配置管理

### 4.2 应用管理模块

#### 4.2.1 菜单管理
- 菜单树形结构展示
- 菜单创建/编辑/删除
- 菜单状态控制（启用/停用）
- 菜单排序和层级管理

#### 4.2.2 权限管理
- 权限列表展示
- 权限创建/编辑/删除
- 权限与菜单关联
- 权限状态控制

#### 4.2.3 测试版配置
- 统一测试版配置管理
- 模块组合配置
- 配额设置

### 4.3 租户管理模块

#### 4.3.1 租户列表
- 租户信息展示（支持搜索和筛选）
- 租户状态管理（待授权/已授权/已停用）
- 租户创建功能

#### 4.3.2 租户详情
- 租户基本信息展示
- 授权状态查看
- NovoGrowth账号信息
- 操作记录时间线

#### 4.3.3 租户授权
- 测试版授权功能
- 授权状态同步到NovoGrowth
- 邮件通知租户

## 5. 非功能性需求

### 5.1 性能要求
- 页面加载时间 < 2秒
- 系统操作响应时间 < 500ms
- 支持100个租户并发使用

### 5.2 安全要求
- 数据传输加密（HTTPS）
- 敏感信息加密存储
- 防止SQL注入和XSS攻击
- 定期安全审计

### 5.3 可用性要求
- 系统可用性 ≥ 99.9%
- 故障恢复时间 < 30分钟
- 数据备份和恢复机制

## 6. 集成接口

### 6.1 NovoGrowth平台集成
- 租户账号创建接口
- 授权配置同步接口
- 账号状态同步接口
- 数据删除接口

### 6.2 邮件服务集成
- 通知邮件发送
- 邮件模板管理
- 邮件发送状态追踪

## 7. 部署环境

### 7.1 开发环境
- Node.js >= 16.x
- npm >= 8.x
- Vite >= 4.x

### 7.2 生产环境
- Nginx >= 1.20
- Docker >= 20.x
- Jenkins >= 2.400

## 8. 项目里程碑

| 阶段 | 时间 | 交付内容 |
|------|------|----------|
| 需求分析 | 2025-10-15 | PRD文档完成 |
| 技术设计 | 2025-10-20 | 技术方案确定 |
| 前端开发 | 2025-11-15 | 前端功能开发完成 |
| 测试验收 | 2025-11-25 | 系统测试通过 |
| 上线部署 | 2025-12-01 | 正式上线运行 |