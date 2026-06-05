# qPCR 仪器预约 H5 网页 — 项目文档

## 一、项目概述

实验室 qPCR 仪器预约系统，从微信小程序转换而来的独立 H5 网页版本。

- **页面地址**：https://siRNA-saRNA.github.io/qpcr-booking-/yuyue.html
- **数据库**：Supabase（PostgreSQL，免费版 500MB）
- **托管**：GitHub Pages（免费）
- **分享图 CDN**：CloudBase（国内加载快）

---

## 二、技术架构

```
┌──────────────────────────────────────────┐
│              用户浏览器                    │
│  ┌────────────────────────────────────┐  │
│  │  yuyue.html (纯前端，零外部依赖)      │  │
│  │  · 原生 JS + CSS + HTML             │  │
│  │  · fetch() 直连 Supabase REST API   │  │
│  │  · localStorage 预填姓名             │  │
│  └──────────────┬─────────────────────┘  │
│                 │  HTTPS REST API        │
│                 ▼                        │
│  ┌────────────────────────────────────┐  │
│  │  Supabase (新加坡)                   │  │
│  │  · instruments — 仪器信息            │  │
│  │  · reservations — 预约记录           │  │
│  │  · settings — 管理员密码             │  │
│  └────────────────────────────────────┘  │
└──────────────────────────────────────────┘
```

**关键决策：**
- 去掉了 Supabase SDK CDN（200KB 阻塞页面），改用原生 `fetch()` 直调 REST API
- 页面先渲染默认数据 → 后台静默加载云端数据 → 刷新
- 北京时间直接取浏览器本地时间

---

## 三、数据库表

### instruments
| 字段 | 类型 | 说明 |
|------|------|------|
| id | UUID | 自动生成 |
| name | TEXT | 如 `7500-1` |
| location | TEXT | 如 `四楼` |
| status | TEXT | `active` / `maintenance` / `broken` |

### reservations
| 字段 | 类型 | 说明 |
|------|------|------|
| id | UUID | 自动生成 |
| user_id | TEXT | 用户标识 |
| instrument_id | UUID | 关联仪器 |
| instrument_name | TEXT | 仪器名称 |
| date | DATE | 预约日期 |
| start_time | TEXT | `HH:mm` |
| end_time | TEXT | `HH:mm` |
| status | TEXT | `active` / `cancelled` |
| name | TEXT | 预约人 |
| notes | TEXT | 备注 |
| booked_at | TIMESTAMP | 预约时间 |

### settings
| 字段 | 类型 | 说明 |
|------|------|------|
| key | TEXT | 主键 |
| value | TEXT | 值 |

---

## 四、业务规则

1. **17:30 规则**：每天 17:30 前所有未来日期时段锁定，17:30 后开放
2. **单仪器限制**：同一用户不能同时预约多台不同仪器
3. **15 天清理**：每次打开页面自动删除 15 天前非活跃记录
4. **管理员密码**：首次点「仪器管理」设密码，之后需密码进入

---

## 五、关键配置

| 配置 | 值 |
|------|-----|
| GitHub 仓库 | `siRNA-saRNA/qpcr-booking-` |
| GitHub Pages | `https://siRNA-saRNA.github.io/qpcr-booking-/yuyue.html` |
| Supabase 项目 | `https://xnayjzwkailupwjsjpvr.supabase.co` |
| Supabase SQL | https://supabase.com/dashboard/project/xnayjzwkailupwjsjpvr/sql/new |
| 本地代码路径 | `D:\桌面\qPCR预约小程序构建\yuyue.html` |

---

## 六、常见修改操作

### 改 17:30 → 其他时间
告诉 Claude：「把 17:30 规则改成 XX:XX」

### 增/删/改仪器
打开 Supabase SQL Editor：
```sql
UPDATE instruments SET name='新名' WHERE name='旧名';
INSERT INTO instruments (name,location,status) VALUES ('7500-8','三楼','active');
```

### 重置管理员密码
```sql
DELETE FROM settings WHERE key='admin_password';
```

### 查看所有预约
Supabase → Table Editor → 选 `reservations` 表

---

## 七、如何与 Claude 沟通修改

### ✅ 有效说法
```
「把 17:30 改成 18:00」
「预约表单加一个邮箱字段」
「手机端按钮太小，调大一点」
「加一个搜索框可以搜仪器名」
```

### ❌ 避免
```
「优化一下」← 太模糊
「这个功能不好」← 没说要怎么改
```

### 操作流程
1. 把上面「关键配置」信息附上
2. 说清楚要改什么
3. Claude 改完 `git push`
4. 打开 `yuyue.html?v=新版本` 验证

### 本地手动推送（网络不好时）
```bash
cd "D:\桌面\qPCR预约小程序构建"
git push
```
