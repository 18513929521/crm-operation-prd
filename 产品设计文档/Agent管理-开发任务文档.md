# Agent 管理 — 开发任务文档（待实现）

| 版本 | 日期 | 说明 |
| --- | --- | --- |
| v1.1 | 2026-08-05 | 菜单改为与作业/任务管理平级（非工具库子菜单） |
| v1.0 | 2026-08-05 | 基于《Agent管理PRD》整理；先文档后实现 |

**归属**：XLeads 子应用（`micro-apps/XLeads-management`）+ 后端 `crm-operation-homework-job`  
**依据**：`Agent管理PRD.md`、`UI图/Agent管理/`  
**关联**：《任务管理-开发任务文档.md》（Agent 执行任务 Step 1 下拉、数据增强场景均依赖本模块）

---

## 一、已确认 / 待确认决策

| # | 议题 | 建议结论 | 状态 |
| --- | --- | --- | --- |
| 1 | 菜单位置 | 侧栏 **XLeads → Agent 管理**，与**任务管理**（原作业管理）、合同列表、订单列表**平级** | **已确认** |
| 2 | 与作业管理关系 | **新增独立模块**；不改造现有 `/job` 作业表 | 已确认方向 |
| 3 | Agent ID 编辑 | **新增可填；编辑不可改**（避免任务引用断裂） | 对齐 PRD |
| 4 | Secret ID 展示 | 列表/详情**全文展示**；后续可做脱敏 | 本期全文 |
| 5 | 默认状态 | 新增保存后默认 **启用** | 对齐 PRD |
| 6 | 删除能力 | **本期不做**物理删除 | 对齐 PRD |
| 7 | 批量启停 | 列表有多选列，**本期不做**批量操作 | 对齐 PRD |
| 8 | Agent 入参 schema | **不在 Agent 管理维护**；执行任务 Step 2 动态表单另接 Agent 平台或配置中心 | 待联调确认 |
| 9 | 后端落库 | 新建 `t_agent` 域，**不改动**现有 `t_homework` | 已确认方向 |

---

## 二、需求范围摘要

### 本期做（P0）

1. Agent 列表：分页、表格内关键词搜索、总数统计  
2. 新增 / 编辑 Agent 弹窗（Agent ID、Agent 名称、Secret ID）  
3. 详情只读弹窗  
4. 启用 / 停用切换（停用二次确认）  
5. 对外 API：分页列表、详情、CRUD、启停；**启用态 Agent 下拉**供任务模块调用  

### 本期不做

| 项 | 说明 |
| --- | --- |
| Secret 脱敏 / 审计日志 | PRD 后续迭代 |
| 批量启停 | 仅预留多选列 |
| Agent schema 配置 UI | 执行任务 Step 2 另需求 |
| 物理删除 | PRD 未提供 |

---

## 三、菜单与路由

### 前端（XLeads-management）

| 菜单名 | 路由 | 完整路径 | 层级 |
| --- | --- | --- | --- |
| Agent 管理 | `/agent` | `/xleads/agent` | XLeads **一级菜单**（平级） |

**XLeads 侧栏平级菜单（参考）**

| 菜单名 | 路由 | 说明 |
| --- | --- | --- |
| 合同列表 | `/contract` | 已有 |
| 订单列表 | `/order` | 已有 |
| 任务管理 | `/task` | 改造原作业管理 `/job` |
| **Agent 管理** | `/agent` | 本期新增 |

> 需在应用管理 / RBAC 中新增 **Agent 管理** 菜单项，`route_path` 指向 `/xleads/agent`；**不需要**「工具库」父级。（PRD 原型写「工具库 → Agent 管理」，本期以本决策为准。）

### 目录建议

```
micro-apps/XLeads-management/src/
├── views/agent/
│   ├── AgentList.vue
│   └── components/
│       ├── AgentForm.vue       # 新增/编辑弹窗
│       └── AgentDetail.vue     # 只读详情弹窗
├── api/modules/
│   └── agent.js
└── constants/
    └── agent.js                # 状态枚举等
```

---

## 四、功能点清单

### A. Agent 列表（P0）

| ID | 功能点 | 说明 |
| --- | --- | --- |
| A-L-01 | 页头 | 标题「Agent 管理」+ **Agent 总数：N 条** |
| A-L-02 | 新增 | 绿色按钮「+ 新增 Agent」→ 打开新增弹窗 |
| A-L-03 | 表格内搜索 | 关键词匹配 Agent ID / 名称 / Secret ID（对齐原型） |
| A-L-04 | 多选列 | Checkbox 预留，本期无批量操作 |
| A-L-05 | 列表列 | Agent ID、Agent 名称、Secret ID、状态、创建时间、操作 |
| A-L-06 | 状态展示 | 启用 / 停用 |
| A-L-07 | 操作-启用 | 编辑、详情、**停用**（红色 link） |
| A-L-08 | 操作-停用 | 编辑、详情、**启用** |
| A-L-09 | 停用确认 | 文案：「停用后新任务将无法选择该 Agent，已有任务不受影响，是否确认停用？」 |
| A-L-10 | 分页 | 底部分页；样式对齐合同/订单列表 `@shared/components/Table` |

### B. 新增 / 编辑 Agent（P0）

| ID | 功能点 | 说明 |
| --- | --- | --- |
| A-F-01 | 弹窗标题 | 新增 Agent / 编辑 Agent |
| A-F-02 | Agent ID | 必填；placeholder「请输入 Agent ID」；编辑态**禁用** |
| A-F-03 | Agent 名称 | 必填；placeholder「请输入 Agent 名称」 |
| A-F-04 | Secret ID | 必填；placeholder「请输入 Secret ID」 |
| A-F-05 | 校验 | 三字段非空；Agent ID 全局唯一 |
| A-F-06 | 保存 | 新增默认 status=启用；关闭弹窗并刷新列表 |
| A-F-07 | 取消 | 不保存关闭 |

### C. Agent 详情（P0）

| ID | 功能点 | 说明 |
| --- | --- | --- |
| A-D-01 | 只读弹窗 | Agent ID、Agent 名称、Secret ID、状态、创建时间、更新时间 |
| A-D-02 | 关闭 | 返回列表 |

### D. 与任务模块联动（P0 接口，任务侧消费）

| ID | 功能点 | 说明 |
| --- | --- | --- |
| A-R-01 | 启用态列表 | `GET /v1/agents/enabled` 或分页带 `status=启用`，供执行任务 Agent 下拉 |
| A-R-02 | 停用拦截 | 任务创建/编辑选用 Agent 时，后端校验必须为启用态 |
| A-R-03 | 已有任务 | 停用**不中断**已引用该 Agent 的在途任务（任务侧保证） |

---

## 五、后端任务（crm-operation-homework-job）

### 5.1 表结构（建议）

**表名：`t_agent`**

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | BIGINT PK | 自增 |
| `agent_id` | VARCHAR(64) UK | Agent ID（业务唯一键，运营录入） |
| `agent_name` | VARCHAR(200) | Agent 名称 |
| `secret_id` | VARCHAR(500) | Secret ID |
| `status` | TINYINT | 1=启用 2=停用 |
| `created_by` / `created_time` | | 审计 |
| `updated_by` / `updated_time` | | 审计 |
| `is_deleted` | TINYINT | 逻辑删除预留 |

DDL 文件建议：`sql/schema/agent_schema.sql`

### 5.2 API 建议

**前缀**：`/v1/agents`（网关 `/homeworkJob/v1/agents`）

| 方法 | 路径 | 说明 |
| --- | --- | --- |
| GET | `/page` | 分页；支持 keyword、status |
| GET | `/enabled` | 启用态全量/分页（任务下拉） |
| GET | `/{id}` | 详情 |
| POST | `/` | 新增 |
| PUT | `/{id}` | 编辑（不可改 agent_id） |
| POST | `/{id}/enable` | 启用 |
| POST | `/{id}/disable` | 停用 |

**响应字段（JSON 驼峰，与前端一致）**

```json
{
  "id": 1,
  "agentId": "qcc-api",
  "agentName": "数据获取",
  "secretId": "v1.0.0",
  "status": 1,
  "statusStr": "启用",
  "createdTime": "2026-08-05 10:00:00",
  "updatedTime": "2026-08-05 10:00:00"
}
```

### 5.3 后端任务拆分

| ID | 任务 | 说明 |
| --- | --- | --- |
| BE-A-01 | DDL + PO/Entity/Enum | `t_agent`、AgentStatus 枚举 |
| BE-A-02 | 分页 / 详情 | keyword 模糊搜三字段 |
| BE-A-03 | 新增 / 编辑 | agent_id 唯一校验 |
| BE-A-04 | 启停 | disable 前无需校验在途任务（业务允许继续跑） |
| BE-A-05 | 启用列表 | 供 Task 模块 Feign 或 HTTP 调用 |

---

## 六、前端任务拆分

| ID | 任务 | 说明 |
| --- | --- | --- |
| FE-A-01 | 路由 + API 模块 | `/agent`、`agent.js` |
| FE-A-02 | AgentList | Table 组件、搜索、操作列 |
| FE-A-03 | AgentForm | 新增/编辑弹窗 |
| FE-A-04 | AgentDetail | 只读详情 |
| FE-A-05 | 常量 | `AGENT_STATUS_OPTIONS` |
| FE-A-06 | 菜单权限 | RBAC 新增 Agent 管理，与任务/合同/订单**平级** |

---

## 七、实现顺序建议

1. **BE-A-01～05** Agent 后端闭环  
2. **FE-A-01～05** Agent 前端页面  
3. **FE-A-06** 菜单配置  
4. 与任务模块联调：执行任务 Agent 下拉仅展示启用态  

> **Agent 模块应优先于任务模块中的「Agent 执行任务」**，否则执行任务 Step 1 无数据源。

---

## 八、联调验收清单

- [ ] 菜单：XLeads 下 Agent 管理与任务管理、合同列表、订单列表**平级**可访问  
- [ ] 新增 Agent：三字段必填；默认启用；列表刷新  
- [ ] Agent ID 重复：新增/编辑保存失败并提示  
- [ ] 编辑：Agent ID 不可改；名称、Secret ID 可改  
- [ ] 停用：二次确认；状态变停用；启用态 API 不再返回该条  
- [ ] 启用：状态变启用；重新出现在启用态 API  
- [ ] 详情：字段只读正确  
- [ ] 样式：列表/弹窗对齐合同订单列表与 UI 原型  

---

## 九、UI 原型对照

| 页面 | 原型路径 |
| --- | --- |
| Agent 列表 | `UI图/Agent管理/Agent管理@1x.png` |
| 编辑弹窗 | `UI图/Agent管理/编辑@1x.png` |

**视觉要点**：页头绿底主按钮「+ 新增 Agent」；表格操作 link 样式；停用按钮红色强调。
