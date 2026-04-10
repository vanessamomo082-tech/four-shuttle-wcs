# 接口流程 / Interface Flow

## 系统接口总览

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   Prime     │      │    IAC     │      │    WES     │
│   (WMS)     │─────▶│  (AGV)     │─────▶│           │
│             │ HTTP │            │  WS  │           │
└─────────────┘      └─────────────┘      └─────────────┘
                                                │
                                                ▼ MQTT
                                        ┌─────────────┐
                                        │    WCS     │
                                        │           │
                                        └─────────────┘
                                                │
                                                ▼ TCP
                                        ┌─────────────┐
                                        │   四穿PLC   │
                                        │           │
                                        └─────────────┘
```

## 入库流程 (Inbound Flow)

### 完整流程步骤

```
1. Prime下单 → IAC: 下发Induction入库任务
              ↓
2. IAC接收任务 → RCS: 一段任务到缓存位
              ↓
3. RCS执行 → 报告完成到IAC
              ↓
4. IAC→WES: 通知站台可用
              ↓
5. WES→IAC: 二段任务放到入库口
              ↓
6. RCS执行 → 报告完成到IAC
              ↓
7. PLC检测 → 光电感应有占位
              ↓
8. PLC→WCS: 通知物料到位
              ↓
9. WCS识别 → WES: 上报BCR信息
              ↓
10. WES判断 → 成功/失败回告Prime
              ↓
11. WCS→PLC: 确认并启动四穿入库
              ↓
12. 四穿执行 → 上架完成
```

### 详细说明

| 步骤 | 操作 | 说明 |
|------|------|------|
| 1 | Prime→IAC | 下发Induction，包含ULID、物料编码、数量 |
| 2 | IAC→RCS | RCS自动调度AGV搬运到缓存位 |
| 3 | RCS→IAC | 报告一段任务完成 |
| 4 | IAC→WES | 通知站台可用于二段搬运 |
| 5 | WES→IAC | 下发二段任务到入库口 |
| 6 | RCS→IAC | 报告二段任务完成 |
| 7 | PLC检测 | 光电开关检测到物料 |
| 8 | PLC→WCS | 发送物料到位信号 |
| 9 | WCS→WES | 上报BCR识别信息 |
| 10 | WES判断 | 成功→继续，失败→异常口 |
| 11 | WCS→PLC | 确认并启动四穿车 |
| 12 | 四穿上架 | 四穿取货→存入指定库位 |

### 入库Induction字段

| 字段 | 说明 | 类型 | 示例 |
|------|------|------|------|
| ULID | 容器ID | String | L001 |
| PRTNUM | 物料编码 | String | PR0201001 |
| PRTQTY | 数量 | Integer | 100 |
| LOC | 推荐库位 | String | ASRS-A-01-01-01 |
| 4WS_DUMMY | 是否需四穿 | Boolean | Y |

## 出库流程 (Outbound Flow)

### 完整流程步骤

```
1. Prime下单 → Pick_Request出库任务
              ↓
2. WES接收 → 分配库存（按托盘号/batch/footprint）
              ↓
3. WES→WCS: 下发下架指令
              ↓
4. WCS调度 → 四穿车取货
              ↓
5. 四穿执行 → 取货到输送带
              ↓
6. WCS→WES: 通知下架完成
              ↓
7. 输送带运输 → 出库口
              ↓
8. 人工/AGV取货 → pick_confirm回传
              ↓
9. WES→Prime: 出库完成确认
```

### 详细说明

| 步骤 | 操作 | 说明 |
|------|------|------|
| 1 | Prime→WES | 下发Pick_Request |
| 2 | WES分配 | 根据策略分配库存 |
| 3 | WES→WCS | 下发下架指令 |
| 4 | WCS调度 | 四穿车取货 |
| 5 | 四穿执行 | 取货→输送带 |
| 6 | WCS→WES | 报告下架完成 |
| 7 | 输送运输 | 物料到出库口 |
| 8 | 取货确认 | 人工/AGV取货并确认 |
| 9 | WES→Prime | 确认出库完成 |

### 出库Pick_Request字段

| 字段 | 说明 | 类型 | 示例 |
|------|------|------|------|
| 托盘号 | 指定托盘 | String | L001 |
| Batch | 批号 | String | LOT20240401 |
| Footprint | 规格 | String | FP001 |
| 数量 | 出库数量 | Integer | 100 |
| 目的地 | 出库口 | Integer | 9 |

### 回传字段

| 字段 | 说明 | 类型 |
|------|------|------|
| pick_confirm | 正常完成 | Success/Failed |
| move_error | 异常回传 | Error Code |

## 接口通信

### Prime-WES接口

#### HTTP REST API

- 方法: POST
- 格式: JSON
- 编码: UTF-8

#### 主要接口

| 接口 | 方法 | 说明 |
|------|------|------|
| /api/items | GET | 同步商品主数据 |
| /api/inventory | GET | 同步库存数据 |
| /api/induction | POST | 下发入库任务 |
| /api/pick_request | POST | 下发出库任务 |
| /api/task_callback | POST | 任务状态回传 |

### WES-IAC接口

#### WebSocket

- 端口: 8080
- 心跳: 30秒

#### 消息格式

```json
{
  "type": "task",
  "action": "assign",
  "data": {
    "task_id": "T001",
    "type": "inbound",
    "from": "AGV01",
    "to": "STATION01"
  }
}
```

### WES-WCS接口

#### MQTT

- Broker: tcp://wcs-server:1883
- Topic: wcs/#
- Qos: 1

#### 订阅主题

| 主题 | 说明 |
|------|------|
| wcs/device/status | 设备状态 |
| wcs/task/status | 任务状态 |
| wcs/alarm | 报警信息 |

#### 发布主题

| 主题 | 说明 |
|------|------|
| wcs/task/inbound | 入库任务 |
| wcs/task/outbound | 出库任务 |

### WCS-PLC接口

#### TCP/IP

- 端口: 5000
- 协议: Modbus TCP

#### 寄存器地址

| 地址 | 说明 | 读写 |
|------|------|------|
| 40000 | 任务指令 | W |
| 40001 | 任务参数 | W |
| 40010 | 设备状态 | R |
| 40011 | 报警状态 | R |
| 40020 | 急停信号 | R |

## 切换流程 (Port Switch)

### 切换条件

1. 现场人员旋转切换按钮
2. 当前任务全部完成
3. PLC检测并上报
4. WCS确认并通知WES
5. WES通知IAC锁闭工作站
6. 切换完成后解锁

### 切换序列

```
1. 现场按钮旋转
     ↓
2. PLC→WCS: 切换请求
     ↓
3. WCS→WES: 切换确认
     ↓
4. WES→IAC: 锁闭工作站
     ↓
5. IAC→WES: 确认锁闭
     ↓
6. WES→WCS: 开始切换
     ↓
7. WCS→PLC: 执行切换
     ↓
8. 切换完成
```

## Prime/IAC接口详解

### Prime接口字段

#### Induction (入库)

| 字段 | 必填 | 说明 |
|------|------|------|
| ulid | ✅ | 容器ID |
| prtnum | ✅ | 物料编码 |
| prtqty | ✅ | 数量 |
| lotnum | ✅ | 批号 |
| ftpcod | ✅ | 托盘规格 |
| mandte | ✅ | 生产日期 |
| invsts | ✅ | 库存状态 |
| expire_dte | - | 过期日期 |

#### Pick_Request (出库)

| 字段 | 必填 | 说明 |
|------|------|------|
| prtnum | ✅ | 物料编码 |
| qty | ✅ | 数量 |
| priority | ✅ | 优先级 |
| location | - | 指定库位 |
| lotnum | - | 指定批号 |
| ftpcod | - | 指定规格 |

###IAC接口字段

#### 任务下发

| 字段 | 说明 |
|------|------|
| task_id | 任务ID |
| task_type | 任务类型(in/out/move) |
| from_loc | 起始位置 |
| to_loc | 目标位置 |
| priority | 优先级 |

#### 状态回传

| 字段 | 说明 |
|------|------|
| task_id | 任务ID |
| status | 状态(new/assigned/done) |
| timestamp | 时间戳 |

## 进出口分配

### 入库工作站

| 入口 | 工作站 | 主要设备 | 备用设备 |
|------|--------|----------|----------|
| 1 | AGV01 | AGV | 叉车 |
| 2 | AGV02 | AGV | 叉车 |
| 3 | AGV03 | AGV | 叉车 |
| 4 | AGV04 | AGV | 叉车 |
| 5 | AGV05 | AGV | 叉车 |
| 6 | AGV06 | AGV | 叉车 |
| 7 | AGV07 | AGV | 叉车 |
| 8 | AGV08 | AGV | 叉车 |
| 9 | 叉车01 | 叉车 | AGV |
| 10 | 叉车02 | 叉车 | AGV |

### 出库工作站

| 出口 | 工作站 | 主要设备 | 备用设备 |
|------|--------|----------|----------|
| 1 | AGV01 | AGV | 叉车/异常 |
| 2 | AGV02 | AGV | 叉车/异常 |
| 3 | AGV03 | AGV | 叉车/异常 |
| 4 | AGV04 | AGV | 叉车/异常 |
| 5 | AGV05 | AGV | 叉车/异常 |
| 6 | AGV06 | AGV | 叉车/异常 |
| 7 | AGV07 | AGV | 叉车/异常 |
| 8 | AGV08 | AGV | 叉车/异常 |
| 9 | 叉车01 | 叉车 | 异常 |
| 10 | 叉车02 | 叉车 | 异常 |

### CFC出库口

| 入口 | 用途 |
|------|------|
| CFC-1 | AGV出库01 |
| CFC-2 | AGV出库02 |
| CFC-3 | AGV出库03 |

## 数据同步

### 同步内容

| 数据类型 | 方向 | 频率 | 触发 |
|----------|------|------|------|
| 商品主数据 | Prime→WES | 实时 | 变更 |
| 库存数据 | Prime→WES | 5分钟 | 定时 |
| 入库任务 | Prime→WES | 实时 | 下单 |
| 出库任务 | Prime→WES | 实时 | 下单 |
| 任务状态 | WES→Prime | 实时 | 变更 |
| 库存差异 | WES→Prime | 30分钟 | 定时 |

### 同步失败处理

| 场景 | 处理 |
|------|------|
| Prime连接失败 | 缓存任务，重连后补推 |
| IAC连接断开 | 自动重连3次 |
| WCS通信中断 | 保持最后状态 |
| PLC无响应 | 多次重试+报警 |