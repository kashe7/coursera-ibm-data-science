# SpaceX Falcon 9 一级火箭着陆预测 — 数据字典

课程目标：用发射特征预测 Falcon 9 **一级助推器能否成功着陆**，从而估计发射成本。  
一行 = 一次发射（或 SQL 实验里的一次有效载荷记录）。

---

## 1. 主数据集（API 采集 + Data Wrangling）

| 项目 | 说明 |
|---|---|
| 来源 | SpaceX REST API `https://api.spacexdata.com/v4/` |
| 课程文件 | `dataset_part_1.csv`（17 列）→ wrangling 后 `dataset_part_2.csv`（18 列，多 `Class`） |
| 规模 | 90 行，全部为 Falcon 9（已去掉 Falcon 1） |
| 粒度 | 每次发射一条记录 |

### 字段

| 字段 | 类型 | 含义 | 取值 / 备注 |
|---|---|---|---|
| `FlightNumber` | int | 过滤 Falcon 1 后重新编号的发射序号 | 1–90 |
| `Date` | str (YYYY-MM-DD) | 发射日期 | 例：`2010-06-04` |
| `BoosterVersion` | str | 火箭型号 | 本表均为 `Falcon 9` |
| `PayloadMass` | float | 有效载荷质量（千克） | 缺失值在 Lab 1 用均值填过；例：约 525–6105 |
| `Orbit` | str | 目标轨道 | 见下方轨道代码 |
| `LaunchSite` | str | 发射场 | `CCAFS SLC 40`、`KSC LC 39A`、`VAFB SLC 4E` |
| `Outcome` | str | 一级着陆结果 = 是否成功 + 着陆方式 | 见下方 Outcome |
| `Flights` | int | 这枚助推器（core）已经飞过几次 | 含本次 |
| `GridFins` | bool | 是否使用网格翼（再入姿态控制） | `True` / `False` |
| `Reused` | bool | 本次是否在用旧助推器 | `True` / `False` |
| `Legs` | bool | 是否使用着陆腿 | `True` / `False` |
| `LandingPad` | str / NaN | 着陆垫 ID | 约 28.9% 缺失（未指定陆地/无人船着陆点） |
| `Block` | float | 一级核心硬件版本 | `1.0`–`5.0`；版本越新通常越容易回收 |
| `ReusedCount` | int | 该核心累计复用次数 | 0 表示从未复用 |
| `Serial` | str | 助推器序列号 | 例：`B0003`、`B1006` |
| `Longitude` | float | 发射场经度 | 例：卡纳维拉尔角约 `-80.58` |
| `Latitude` | float | 发射场纬度 | 例：卡纳维拉尔角约 `28.56` |
| `Class` | int | **训练标签**（仅 `dataset_part_2.csv`） | `1` 成功着陆，`0` 失败或未尝试 |

`Class` 由 `Outcome` 生成：若 `Outcome` 属于失败集合则为 0，否则为 1。

```python
bad_outcomes = {'False ASDS', 'False Ocean', 'False RTLS', 'None ASDS', 'None None'}
landing_class = [0 if outcome in bad_outcomes else 1 for outcome in df['Outcome']]
df['Class'] = landing_class
```

### `Outcome` 取值

格式：`{landing_success} {landing_type}`。

| 值 | 含义 | 计入 Class | 本数据集条数 |
|---|---|---|---|
| `True ASDS` | 成功降落无人船 | 1 | 41 |
| `True RTLS` | 成功降落陆地回收场 | 1 | 14 |
| `True Ocean` | 成功受控降落海面指定区域 | 1 | 5 |
| `False ASDS` | 尝试无人船降落，失败 | 0 | 6 |
| `False RTLS` | 尝试陆地降落，失败 | 0 | 1 |
| `False Ocean` | 尝试海面降落，失败 | 0 | 2 |
| `None ASDS` | 计划无人船但未完成着陆 | 0 | 2 |
| `None None` | 未尝试回收 | 0 | 19 |

缩写：

- **ASDS**：Autonomous Spaceport Drone Ship（无人船）
- **RTLS**：Return To Launch Site（返回发射场陆地垫）
- **Ocean**：海上受控降落

### `Orbit` 取值

| 代码 | 含义 |
|---|---|
| `LEO` | 近地轨道（Low Earth Orbit） |
| `VLEO` | 极近地轨道（Very Low Earth Orbit，约 &lt; 450 km） |
| `ISS` | 国际空间站轨道 |
| `PO` | 极地轨道（Polar Orbit） |
| `SSO` / `SO` | 太阳同步轨道 |
| `GTO` | 地球同步转移轨道（Geostationary Transfer Orbit） |
| `GEO` | 地球同步轨道 |
| `MEO` | 中地球轨道 |
| `HEO` | 大椭圆轨道 |
| `ES-L1` | 日地拉格朗日 L1 点 |

### `LaunchSite` 取值

| 代码 | 地点 |
|---|---|
| `CCAFS SLC 40` | 卡纳维拉尔角 SLC-40（Cape Canaveral） |
| `KSC LC 39A` | 肯尼迪航天中心 LC-39A |
| `VAFB SLC 4E` | 范登堡 SLC-4E（加州） |

---

## 2. SQL 实验数据集（`Spacex.csv` / 表 `SPACEXTBL`）

与主预测表不是同一份文件：更偏“任务/载荷”记录，字段名和着陆结果写法都不同。

| 项目 | 说明 |
|---|---|
| 来源 | [Spacex.csv](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/labs/module_2/data/Spacex.csv) |
| 表名 | `SPACEXTBL` → 去掉空日期后为 `SPACEXTABLE` |
| 规模 | 101 行 × 10 列 |

| 字段 | 类型 | 含义 |
|---|---|---|
| `Date` | str | 发射日期 |
| `Time (UTC)` | str | 发射时间（UTC） |
| `Booster_Version` | str | 助推器版本与序列号，例：`F9 v1.0  B0003` |
| `Launch_Site` | str | 发射场，例：`CCAFS LC-40` |
| `Payload` | str | 载荷名称/描述 |
| `PAYLOAD_MASS__KG_` | int | 载荷质量（千克） |
| `Orbit` | str | 目标轨道，写法可能带括号，例：`LEO (ISS)` |
| `Customer` | str | 客户，例：NASA、SpaceX |
| `Mission_Outcome` | str | **任务**是否成功（把卫星送入轨道），不是一级着陆 |
| `Landing_Outcome` | str | 一级着陆结果（英文短语，见下） |

列名含空格或大小写混用时，SQL 里要用双引号，例如 `"Landing_Outcome"`。

### `Landing_Outcome` 取值（SQL 表）

| 值 | 含义 |
|---|---|
| `Success (drone ship)` / `Success` | 成功（无人船或后期统一写法） |
| `Success (ground pad)` | 成功降落陆地垫 |
| `Controlled (ocean)` | 受控海面降落 |
| `Uncontrolled (ocean)` | 不受控海面降落 |
| `Failure (drone ship)` / `Failure` / `Failure (parachute)` | 着陆失败 |
| `Precluded (drone ship)` | 未能进行无人船着陆 |
| `No attempt` / `No attempt ` | 未尝试回收（注意有一条带尾部空格） |

不要把 `Mission_Outcome` 和 `Landing_Outcome` 搞混：前者是任务成败，后者才是一级火箭回收。

---

## 3. 对应实验

| 实验 | 笔记本 | 使用的数据 |
|---|---|---|
| Lab 1 数据采集 | `jupyter-labs-spacex-data-collection-api.ipynb` | SpaceX API → 主表 17 列 |
| Lab 2 Data Wrangling | `labs-jupyter-spacex-Data wrangling-v2.ipynb` | `dataset_part_1.csv` → 增加 `Class` |
| SQL EDA | `jupyter-labs-eda-sql-coursera_sqllite.ipynb` | `Spacex.csv` / `SPACEXTABLE` |
| Folium 发射场地图 | `lab_jupyter_launch_site_location.ipynb` | `spacex_launch_geo.csv` → `spacex_df` |

---

## 4. Folium 实验数据集（`spacex_df` / `spacex_launch_geo.csv`）

| 项目 | 说明 |
|---|---|
| 来源 | [spacex_launch_geo.csv](https://cf-courses-data.s3.us.cloud-object-storage.appdomain.cloud/IBM-DS0321EN-SkillsNetwork/datasets/spacex_launch_geo.csv) |
| 规模 | 56 行 × 13 列（读入后） |
| 粒度 | 每次发射一条记录 |
| 和主表的区别 | 行更少（到约 2018）、列名带空格；多了 `Lat` / `Long`，用 `class` 而不是 `Class` |

### 读入时的全部字段

| 字段 | 类型 | 含义 |
|---|---|---|
| `Flight Number` | int | 发射序号，1–56 |
| `Date` | str | 发射日期，`2010-06-04`–`2018-06-04` |
| `Time (UTC)` | str | 发射时间（UTC） |
| `Booster Version` | str | 助推器版本与序列号，例：`F9 v1.0  B0003` |
| `Launch Site` | str | 发射场（带空格，不是 `LaunchSite`） |
| `Payload` | str | 载荷名称 |
| `Payload Mass (kg)` | float | 载荷质量（千克），0–9600 |
| `Orbit` | str | 目标轨道，例：`LEO`、`LEO (ISS)`、`GTO`、`Polar LEO` |
| `Customer` | str | 客户 |
| `Landing Outcome` | str | 一级着陆结果（英文短语，空格可能不整齐） |
| `class` | int | **着陆是否成功**：`1` 成功，`0` 失败/未成功 |
| `Lat` | float | 发射场纬度 |
| `Long` | float | 发射场经度 |

### 笔记本里实际用来画图的列

读入后通常会收成这 4 列，Task 2 再加颜色：

| 字段 | 类型 | 含义 |
|---|---|---|
| `Launch Site` | str | 发射场 |
| `Lat` | float | 纬度 |
| `Long` | float | 经度 |
| `class` | int | 1 = 成功，0 = 失败 |
| `marker_color` | str | Task 2 生成：`class==1` → `green`，否则 `red` |

`launch_sites_df` 是按 `Launch Site` 去重后只留坐标，4 行：

| Launch Site | 地点 | Lat | Long |
|---|---|---|---|
| `CCAFS LC-40` | 卡纳维拉尔角 LC-40 | 约 28.56 | 约 -80.58 |
| `CCAFS SLC-40` | 卡纳维拉尔角 SLC-40（同场升级） | 约 28.56 | 约 -80.58 |
| `KSC LC-39A` | 肯尼迪航天中心 39A | 约 28.57 | 约 -80.65 |
| `VAFB SLC-4E` | 范登堡（加州） | 约 34.63 | 约 -120.61 |

列名带空格时用 `df['Launch Site']`，不要写成 `LaunchSite`。`class` 全小写，和 wrangling 表里的 `Class` 不是同一份数据。
