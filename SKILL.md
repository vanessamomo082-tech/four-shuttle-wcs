---
name: four-shuttle-wcs
description: |
  宝洁黄埔四项穿梭车立体库WCS系统知识技能。适用于：(1)四项穿梭车立体库的库存管理问题(2)处理出入库任务异常(3)理解与WES/AGV的接口对接逻辑(4)查询业务解析配置(5)了解闲时理货逻辑
---

# 四项穿梭车立体库 WCS 系统 / Four-Way Shuttle WCS System

## 项目概述 / Project Overview

- **客户 / Customer**: 宝洁黄埔工厂 / P&G Huangpu Plant
- **系统 / System**: WCS (Warehouse Control System) 控制四项穿梭车
- **上游 / Upstream**: Prime (WMS) 和 IAC (AGV 调度)
- **下游 / Downstream**: 四穿 PLC 控制系统
- **总库位 / Total Locations**: 1706个
- **储存层数 / Storage Levels**: 4层
- **进出口 / Ports**: 10个WH口 + 3个CFC口

## 核心概念 / Core Concepts

### 名词解释 / Terminology

| 术语 / Term | 说明 / Description |
|-------------|---------------------|
| Prime 排位 | 上游Prime记录库存的单位 |
| 地台板条码 | 对应WES的容器编号 (Load Number) |
| 四穿库 | 四项穿梭车立体库（双向穿梭车）|
| Induction | 入库任务单 |
| Pick_Request | 出库任务单 |
| ULID | 容器ID |
| BCP | 暂存排位 |
| 4WS_DUMMY | 需要进四穿库的标记 |

### 货位结构 / Location Structure

- **深位组 / Depth Group**: 6深位库位组（每组可存6托）
- **层 / Level**: 优先推荐层配置
- **库位编码 / Location Code**: 格式 ASRS-区域-排-层-位

## 核心功能 / Core Functions

### 1. 库存管理 / Inventory Management

- 货位管理（货架、货位）
- 容器管理（容器类型、容器库存）
- 物料类型管理
- 批次属性（LOTNUM, FTPCOD, MANDTE, INVSTS, EXPIRE_DTE）
- 商品信息（来源Prime）

### 2. 任务管理 / Task Management

- 入库任务：Prime下发Induction → WES生成上架任务 → WCS调度四穿
- 出库任务：Prime下发Pick_Request → WES分配库存 → WCS调度下架
- 任务状态：新建 → 受理 → 执行 → 完成/失败
- 任务优先级：1-50级，数值越低优先级越高

### 3. 策略配置 / Strategy Configuration

- **上架策略 / Putaway Strategy**: 同SKU同批次优先 → 空位组最深层 → 异常
- **分配策略 / Allocation Strategy**: 按Prime要求 + WES最优
- **倒库策略 / Reallocation Strategy**: 外深位托盘先出
- **优先级升级 / Priority Upgrade**: 范围1-50，每10分钟降2级

## 系统架构 / System Architecture

### 层级结构 / Layer Structure

```
Prime (WMS) → IAC (AGV调度) → WES (仓储执行) → WCS (设备控制) → 四穿PLC → 设备
```

### 通信接口 / Communication Interfaces

| 接口 / Interface | 协议 / Protocol | 说明 / Description |
|-----------------|------------------|---------------------|
| Prime-WES | HTTP/API | 商品/库存同步 |
| WES-IAC | WebSocket | 任务下发 |
| WES-WCS | MQTT | 设备控制 |
| WCS-PLC | TCP/IP | PLC通信 |

### 数据同步 / Data Sync

| 数据类型 | 同步频率 |
|----------|----------|
| 商品数据 | 实时 |
| 库存数据 | 5分钟/次 |
| 任务状态 | 实时 |
| 库存差异 | 30分钟/次 |

---

## 异常处理方案 / Exception Handling

### 入库异常 / Inbound Exceptions

| 异常 / Exception | 处理 / Solution |
|-----------------|------------------|
| BCR扫码异常 | 输送线回退 → 异常口 |
| 超高超宽 | 输送线回退 → 异常口 |
| 物料类型不允许入四穿 | 转BCP排位，检查item family |
| 没有可用库位 | 转入库旁地面排位 |
| 四穿库内存在相同托盘号 | 转地面排位，调查差异 |
| 商品数量大于板规 | 转地面排位，维护footprint |
| 入库时卡托盘 | 复位/��制失败/手动完成 |

### 出库异常 / Outbound Exceptions

| 异常 / Exception | 处理 / Solution |
|-----------------|------------------|
| 质量状态变更 | 继续执行 → 回传move_error |
| 取消失败 | 继续执行 → 回传move_error |
| 没有匹配库存 | 检查Prime库存比对 |

### 任务状态处理 / Task Status

| 任务状态 | 可取消 | 可修改优先级 | 可强制完成 |
|----------|--------|-------------|------------|
| 新建 / New | ✅ | ✅ | ✅ |
| 受理 / Accepted | ❌ | ❌ | ✅ |
| 执行 / Executing | ❌ | ❌ | ✅ |
| 完成 / Completed | ❌ | ❌ | ❌ |
| 失败 / Failed | ❌ | ❌ | ❌ |

### 现场异常处理（OPL）/ Site OPL

| 异常问题 | 操作指引 / Operation |
|----------|---------------------|
| 外检异常 | 严重：拉回车间整理；不严重：叉车司机简单整理后重新扫描入库 |
| 无上架任务 | 确认排位是否ASRS进口排位，重新扫描入库 |
| 无条码信息 | 条码破损，重新换条码 |
| 系统显示ASRS_DUMMY | 取消供线任务，重新扫描入库 |
| 商品分类不允许入四穿 | 材料转回正常排位储存 |

### 出入口操作要求 / Port Operation

| 指示灯 | 状态 | 操作 |
|--------|------|------|
| 红色 | 出库状态 | 不允许放货 |
| 绿色 | 入库状态 | 可以放货 |
| 红绿闪烁 | 取货提醒 | 有货送出，可取走 |

---

## 操作流程 / Operations

### 收货检查流程 / Receiving Inspection

**场景1：整板送货+供应商标签**
1. 卸货划掉供应商ULID
2. 检查超板/缠膜 → 超板需整理，拒收
3. 检查地台板破损 → 拒收贴"out of usu"
4. 检查条码完整性 → 缺则贴4联标签
5. 收货使用ULID
6. 上库前检查底部 → 破损需换板
7. 上架用"定向" → 4WS_DUMMY送立库9/10口

### 入库操作 / Inbound Operation

1. 材料收货后上架，按F6键
2. 选择"1 Directed（定向工作）"
3. 系统自动推荐排位
4. 4WS_DUMMY = 需要进四穿库
5. 叉车到入库口（指示灯绿色）
6. RDT扫描入货口排位
7. 放输送带完成

### 出库操作 / Outbound Operation

1. Prime下Pick_Request（有托盘号则严格按托盘分配）
2. WES分配库存（无托盘号按batch和footprint分配）
3. WCS调度下架
4. 输送带送到出库口
5. 叉车取货

### 指定容器出库 / Container Picking

1. WES系统打开指定容器出库
2. 输入Load Number，按查询
3. 选择出库口（01=9#口，02=10#口）
4. 按保存（出库口需手动改为出库状态）
5. 任务列表显示任务状态

### 退料操作 / Returns

1. 生产线退料前确认材料外形、标签、地台板不超板
2. 确认标签与实物一致
3. 用原供应商板（或仓库板）
4. 数量不能大于Prime叠堆数量
5. 上架用"定向"，4WS_DUMMY送立库9/10口

---

## WES系统维护 / WES System Maintenance

### 设置item family是否可以储存

- WES系统 → 商品类型 → 选择item family → 编辑 → 是否四穿库储存选择"是/否"

### 设置材料储存优先级

- WES系统 → 商品类型 → 选择item family → 编辑 → 优先推荐层选择层数

### 任务>30min处理

1. 登录WES任务界面
2. 滞留时间选择">30min"，点击查询
3. 调查Prime库存和操作记录，核对库存位置
4. 状态=新建：不需要执行则WES取消任务
5. 状态=执行：不需要执行则WES强制完成任务

---

## 运营指标 / Performance Metrics

| 指标 | 目标值 | 说明 |
|------|--------|------|
| 入库效率 | 30托/小时 | 单口入库能力 |
| 出库效率 | 40托/小时 | 单口出库能力 |
| 设备可用率 | 95% | 四穿运行时间占比 |
| 任务及时率 | 98% | 30分钟内完成 |

### 容量指标

| 指标 | 数值 |
|------|------|
| 总库位 | 1706 |
| 当��库存 | 动态 |
| 空库位 | 动态 |
| 利用率 | 动态 |

---

## 维护保养 / Maintenance

### 日常维护

- 每班检查输送带运行状态
- 每班检查BCR扫码灵敏度
- 每日清洁传感器
- 每周给导轨加润滑油

### 定期保养

| 周期 | 内容 |
|------|------|
| 每月 | 全面检查设备运行状态 |
| 每季 | 更换磨损配件 |
| 每年 | 大修维护 |

---

## 设备参数 / Equipment Parameters

| 参数 | 值 |
|------|------|
| 总库位 | 1706 |
| 储存层数 | 4 |
| 深度 | 1-6深 |
| 货架编码 | HPRPM-01/02/03/04 |
| WH进出口 | 10个(1-10) |
| CFC进出口 | 3个(1-3) |

---

## 设备故障排查 / Troubleshooting

### 常见故障

| 故障 | 可能原因 | 解决方法 |
|------|----------|----------|
| 四穿不运行 | 电源/控制柜 | 检查急停/电源 |
| 输送带不转 | 电机/变频器 | 检查驱动/电机 |
| BCR不响应 | 条码损坏/镜头脏 | 清洁镜头/换条码 |
| PLC报警 | 传感器异常 | 检查传感器 |
| 任务卡住 | 通信中断 | 重启通信 |

---

## 日常检查项 / Daily Check

### 叉车司机检查

- 检查地台板4面条码完整性
- 检查材料/缠膜不超板
- 检查标签不翘起、无划痕
- 上架前检查地台板底部

### 仓库DA检查

- 核对实物与标签信息一致
- 检查地台板是否破损
- 检查缠膜是否符合212标准

### WES操作员检查

- 检查任务滞留时间>30min
- 核对Prime与WES库存一致性
- 检查设备异常报警
- 任务强制完成后核对库存

---

## 常见问题FAQ

**Q: 4WS_DUMMY是什么意思？**
A: 代表材料需要进四穿库储存，系统会推荐入库口9/10

**Q: 任务为什么被取消？**
A: 新建状态可取消，执行中需强制完成或检查设备

**Q: 入库口指示灯红色可以放货吗？**
A: 不可以，红色代表出库状态

**Q: 如何查询库存位置？**
A: 在WES系统-库存界面输入Load Number查询

**Q: 任务>30min未完成怎么办？**
A: 登录WES查询滞留时间，检查原因后取消或强制完成

---

## 参考资料 / References

- [references/business-rules.md](references/business-rules.md) - 业务规则详解 / Business Rules
- [references/interface-flow.md](references/interface-flow.md) - 接口流程图 / Interface Flow
- [references/error-codes.md](references/error-codes.md) - 异常代码 / Error Codes