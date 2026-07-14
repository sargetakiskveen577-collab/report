# 东方智学 · 库存管理系统

公司图书、电子产品、礼品、项目物料的统一库存管理 Web 应用。支持实时出入库看板、资产库存管理、申请领用（提交即出库）、申请记录、出入库记录与统计看板。无审批流，提交申请即生成完整出库记录并扣减库存。

## 技术栈

- 后端：Node.js + Express + better-sqlite3（零配置嵌入式数据库）+ JWT 鉴权
- 前端：React + TypeScript + Vite + Ant Design 5 + ECharts
- 单仓库前后端，生产构建后由后端统一托管静态资源

## 快速开始

```bash
# 1. 安装依赖
cd server && npm install
cd ../client && npm install

# 2. 构建前端
cd client && npm run build

# 3. 启动后端（自动托管前端 dist，默认端口 4000）
cd ../server && npm start
# 或开发模式（前后端分离热更新）：
#   终端1: cd server && npm run dev
#   终端2: cd client && npm run dev
```

访问 http://localhost:4000

## 默认账号

| 用户名 | 密码     | 角色   |
|--------|----------|--------|
| admin  | admin123 | 管理员 |

管理员可管理资产、手动出入库、管理用户；普通用户可查看库存、提交申请、查看各类记录。

## 功能说明

- **看板总览**：资产种类、库存总量、今日出入库、待归还、缺货预警、近30天趋势、类别/城市分布。
- **资产库存**：按类别（设备/图书/礼品/项目物料）、城市、关键词筛选；支持新增/编辑/删除。
  - 设备按单品管理（有资产系统编码）；图书/礼品/物料按数量管理。
- **申请领用**：选择资产 → 填写数量/申请人/用途 → 提交即出库并自动扣减库存，生成申请记录与出库记录。
  - 支持「是否扣减库存」「是否需要归还」；需归还的可在申请记录页一键归还，库存自动恢复。
- **申请记录**：所有领用申请，含归还状态。
- **出入库记录**：全部出入库流水（申请出库、归还入库、采购入库、资产变更、手动调整），支持筛选与手动调整（管理员）。
- **用户管理**：管理员维护账号、角色、城市。

## 数据迁移

已将从飞书多维表格「东方智学公司27财年资产统计表」的 9 张表迁移至本系统：

| 飞书表 | 映射到 |
|--------|--------|
| 设备清单 (116) | 设备（单品） |
| 书柜存书 (344) / 教研教材 (250) / 样品书 (28) | 图书（数量） |
| 礼品清单 (31) | 礼品（数量） |
| 项目物料 (33) | 项目物料（数量） |
| 固定资产变更记录 (47) / 礼品使用记录 (6) | 出入库记录（历史） |

图书使用记录表为空，未迁移。共迁移 **802 条资产** 与 **53 条历史出入库记录**。

重新迁移（需先停后端）：

```bash
cd server
# 确保 /tmp/feishu_raw.json 存在（由抓取脚本生成）
NODE_PATH=./node_modules node ../scripts/migrate.js
```

## 目录结构

```
inventory-system/
├── server/
│   ├── src/
│   │   ├── db.js          # 数据库初始化与建表
│   │   ├── auth.js        # JWT 鉴权中间件
│   │   ├── utils.js       # 工具函数
│   │   ├── index.js       # Express 入口（托管前端 dist）
│   │   └── routes/        # auth / assets / inventory / requests / stats / users
│   └── data/inventory.db  # SQLite 数据库（自动生成）
├── client/
│   └── src/
│       ├── api.ts  constants.ts  App.tsx
│       └── pages/  # Login / Dashboard / Assets / Apply / Requests / Records / Users
└── scripts/migrate.js      # 飞书数据迁移脚本
```

## 部署

生产环境构建前端后，仅需运行后端即可（`server/data/inventory.db` 为数据文件，做好备份）。可配合 Nginx 反向代理或 pm2 守护进程常驻。
