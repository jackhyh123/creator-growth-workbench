# 反向海淘 · 达人增长工作台

一个单文件、纯前端的达人（KOL/网红）合作增长工作台。面向做**反向海淘**的运营团队：
从达人线索、建联、寄样、内容发布，到包裹物流异常、投产比核算，全部收在一个 HTML 文件里。

- 无需后端、无需构建，双击 `index.html` 就能用
- 数据保存在浏览器本地（localStorage），也可以接自己的接口
- 自带**脱敏演示数据**：14 位达人、15 个包裹、若干待办与工单，打开即可体验

> 仓库里的所有数据均为演示用虚构数据，不含任何真实业务信息。

## 快速开始

```bash
git clone https://github.com/<your-name>/creator-growth-workbench.git
cd creator-growth-workbench
open index.html          # macOS
# 或直接拖进浏览器
```

首次打开是演示数据；自己的数据请按下面的「数据结构」接入。

## 解决什么问题

| 场景 | 工作台的回答 |
|---|---|
| 达人太多，不知道先跟谁 | 合作阶段分桶 + 待处理事项，按优先级排队 |
| 寄了样品，包裹到哪了 | 包裹页：运输阶段、状态、提醒三层视图 |
| 物流出问题没人管 | 异常/待核查自动归类：明确异常、7 天无更新、查不到轨迹、运单号待补 |
| 谁该付钱、谁还没付 | 包裹状态接后台真实状态：待付款／补款、取消／退回 |
| 投入产出算不清 | 投入产出页：成本、GMV、产出比 |

## 核心概念

**两层筛选（包裹页）**

1. 一级：**运输阶段** —— 包裹走到哪一步（仓内待出库 / 国内待起飞 / 国际运输中 / 海关处理中 / 清关后派送 / 已签收）
2. 二级：**包裹状态** —— 这个包裹有没有问题（正常 / 已签收 / 待付款／补款 / 取消／退回 / 已过期 / 运单号待补 / 查不到轨迹 / 状态待确认 / 异常）

二级标签会随一级收敛：选了「仓内待出库」，就只显示这个阶段里真实存在的状态和数量。
标签支持**自定义**：勾选控制显隐、按住右侧 ⠿ 拖动排序，配置存在本地。

**状态判定规则**见 [docs/parcel-rules.md](docs/parcel-rules.md)。

## 数据结构

页面里只有一个全局对象 `const DATA = {...}`，直接从后端接口生成即可：

```jsonc
{
  "today": "2026-11-22",
  "creators": [{
    "id": "c1",
    "name": "达人名",
    "handle": "tiktok_handle",
    "email": "demo@example.com",
    "owner": "负责人",
    "stage": "outreach",
    "parcels": [{
      "pid": "S2026DEMO000001",
      "track": "DEMO000000001CN",
      "status": "待支付",
      "trk": "最新一条轨迹文案",
      "trkStatus": "运输中",
      "trkSub": "Exception_Security",
      "trkAt": "2026-11-12 10:26",
      "lastCheckedAt": "2026-11-23 10:41",
      "scmLabel": "待支付"
    }],
    "tickets": [{ "t": "跟进话术", "cat": "沟通" }]
  }],
  "tasks": [], "tickets": [], "owners": [], "tiers": [],
  "pcount": {}, "coopcount": {}, "n_creators": 14, "syncedAt": {}
}
```

字段说明：

| 字段 | 含义 |
|---|---|
| `stage` | 合作阶段：待确认 / 建联 / 寄样 / 已发布 / 持续合作 / 暂停 |
| `parcels[].status` | 订单侧原始状态文案（用于兜底判断） |
| `parcels[].trkStatus` / `trkSub` | 物流侧主状态 / 子状态（枚举见规则文档） |
| `parcels[].scmLabel` | 后台真实包裹状态（待支付、待发货…） |
| `parcels[].track` | 运单号，空字符串表示还没出单号 |

## 接入自己的数据

工作台不绑定任何一家服务，推荐两种接法：

1. **生成静态 DATA**：定时任务从自己的后台拉数据，替换 `index.html` 里 `const DATA = {...}` 后发布。本仓库的演示数据就是这么来的。
2. **接口直连**：把 `const DATA = {...}` 换成 `fetch('/api/workbench').then(r=>r.json())`，其余渲染逻辑不变。

物流轨迹可接 [17TRACK API v2.4](https://api.17track.net)，注册单号后按批查询；接入要点见 [docs/integrations.md](docs/integrations.md)。

## 目录

```
index.html              工作台（含演示数据）
docs/parcel-rules.md    包裹状态与提醒判定规则
docs/integrations.md    物流/后台数据接入说明
```

## 更新方式

- 本仓库是**公开镜像**：只包含界面、规则和演示数据
- 内部版本更新后，重新导出并覆盖 `index.html`；接口与规则改动同步更新 `docs/`
- 任何密钥、账号、真实订单、真实达人信息都不会进入本仓库

## License

MIT
