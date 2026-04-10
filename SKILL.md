---
name: four-shuttle-wcs
description: |
  P&G Huangpu Four-Way Shuttle立体库WCS系统知识技能。
  适用于：(1)四项穿梭车立体库的库存管理问题(2)处理出入库任务异常(3)理解与WES/AGV的接口对接逻辑(4)查询业务解析配置(5)了解闲时理货逻辑
---

# 四项穿梭车立体库 WCS 系统 / Four-Way Shuttle WCS System

## 项目概述 / Project Overview

| 项目 / Item | 内容 / Content |
|-------------|----------------|
| 客户 / Customer | 宝洁黄埔工厂 / P&G Huangpu Plant |
| 系统 / System | WCS (Warehouse Control System) |
| 上游 / Upstream | Prime (WMS) + IAC (AGV调度 / AGV Dispatch) |
| 下游 / Downstream | 四穿PLC控制系统 / Four-Way Shuttle PLC |
| 总库位 / Total Locations | 1706个 / 1706 locations |
| 储存层数 / Storage Levels | 4层 / 4 levels |
| 进出口 / Ports | 10个WH口 + 3个CFC口 |

---

## 核心概念 / Core Concepts

### 名词解释 / Terminology

| 术语 / Term | 说明 / Description |
|-------------|---------------------|
| Prime 排位 / Prime Location | 上游Prime记录库存的单位 / Prime inventory unit |
| 地台板条码 / Pallet Barcode | 对应WES的容器编号 / Load Number |
| 四穿库 / Four-Way Shuttle | 四项穿梭车立体库 / Bi-directional shuttle |
| Induction | 入库任务单 / Inbound task |
| Pick_Request | 出库任务单 / Outbound task |
| ULID | 容器ID / Container ID |

### 货位结构 / Location Structure

- **深位组 / Depth Group**: 6深位库位组（每组可存6托）/ 6-deep location groups (6 pallets each)
- **层 / Level**: 优先推荐层配置 / Priority level configuration
- **库位编码 / Location Code**: 格式 / Format: ASRS-区域-排-层-位

---

## 核心功能 / Core Functions

### 1. 库存管理 / Inventory Management

- 货位管理 / Location management
- 容器管理 / Container management
- 物料类型管理 / Item family management
- 批次属性 / Batch attributes (LOTNUM, FTPCOD, MANDTE, INVSTS, EXPIRE_DTE)

### 2. 任务管理 / Task Management

- 入库 / Inbound: Prime → Induction → WES → WCS → Shuttle
- 出库 / Outbound: Prime → Pick_Request → WES allocation → WCS → Shuttle

### 3. 策略配置 / Strategy Configuration

- **上架策略 / Putaway Strategy**: 同SKU同批次 → 空位组最深层 → 异常
- **分配策略 / Allocation Strategy**: 按Prime要求 + WES最优
- **倒库策略 / Reallocation Strategy**: 外深位托盘先出

---

## 系统架构 / System Architecture

```
Prime (WMS) → IAC (AGV) → WES → WCS → PLC → Equipment
```

### 通信接口 / Communication Interfaces

| 接口 / Interface | 协议 / Protocol | 说明 / Description |
|-----------------|------------------|---------------------|
| Prime-WES | HTTP/API | 商品/库存同步 |
| WES-IAC | WebSocket | 任务下发 |
| WES-WCS | MQTT | 设备控制 |
| WCS-PLC | TCP/IP | PLC通信 |

---

## 异常处理 / Exception Handling

### 入库异常 / Inbound Exceptions

| 异常 / Exception | 处理 / Solution |
|-----------------|------------------|
| BCR扫码异常 | 输送线回退 → 异常口 |
| 超高超宽 | 输送线回退 → 异常口 |
| 物料类型不允许 | 转BCP排位 |

### 出库异常 / Outbound Exceptions

| 异常 / Exception | 处理 / Solution |
|-----------------|------------------|
| 质量状态变更 | 继续执行 → 回传move_error |
| 取消失败 | 继续执行 → 回传move_error |

### 任务状态 / Task Status

| 状态 / Status | 可取消 / Cancelable | 可改优先级 / Priority Editable |
|---------------|---------------------|--------------------------------|
| 新建 / New | ✅ | ✅ |
| 受理 / Accepted | ❌ | ❌ |
| 执行 / Executing | ❌ | ❌ |
| 完成 / Completed | ❌ | ❌ |

---

## 操作流程 / Operations

### 入库操作 / Inbound Operation

1. 按F6键 / Press F6
2. 选择"1 Directed" / Select "1 Directed"
3. 系统推荐排位 / System recommends location
4. 4WS_DUMMY = 需要进四穿库
5. 叉车到入库口（绿灯） / Forklift to inbound port (green light)
6. RDT扫描入库口 / RDT scan inbound port
7. 放输送带完成 / Place on conveyor to complete

### 出库操作 / Outbound Operation

1. Prime下Pick_Request
2. WES分配库存 / WES allocates inventory
3. WCS调度下架 / WCS dispatches retrieval
4. 输送带送到出库口 / Conveyor to outbound port
5. 叉车取货 / Forklift picks up

---

## 设备参数 / Equipment Parameters

| 参数 / Parameter | 值 / Value |
|------------------|------------|
| 总库位 / Total Locations | 1706 |
| 储存层数 / Storage Levels | 4 |
| 深度 / Depth | 1-6深 |
| 货架编码 / Rack Code | HPRPM-01~04 |
| WH进出口 / WH Ports | 10 (1-10) |
| CFC进出口 / CFC Ports | 3 (1-3) |

---

## 效率指标 / Performance Metrics

| 指标 / Metric | 目标 / Target |
|---------------|---------------|
| 入库效率 / Inbound Efficiency | 30托/小时 |
| 出库效率 / Outbound Efficiency | 40托/小时 |
| 设备可用率 / Equipment Availability | 95% |
| 任务及时率 / Task On-time Rate | 98% |

---

## 参考资料 / References

- references/business-rules.md - 业务规则 / Business Rules
- references/interface-flow.md - 接口流程 / Interface Flow
- references/error-codes.md - 异常代码 / Error Codes
