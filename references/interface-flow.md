# 接口对接流程 / Interface Flow

## 入库流程 / Inbound Flow

### 整体流程 / Overview

```
Prime → IAC WCS → 快仓 RCS → 缓存位 → 四穿入库口 → WES → WCS → 四穿 → 上架
```

### 详细步骤 / Steps

| 步骤 | 接口 | 描述 |
|------|------|------|
| 1 | Prime→IAC | 下发去四穿库的 Induction |
| 2 | IAC→RCS | 下发一段任务到缓存位 |
| 3 | RCS→IAC | 搬运完成回告 |
| 4 | WES→IAC | 发送作站台是否可用 |
| 5 | IAC→RCS | 下发二段任务放到四穿入库口 |
| 6 | RCS→IAC | AGV托盘放下回告 |
| 7 | PLC→WCS | 光电感应入库口有占位 |
| 8 | WCS→WES | 发送设备识别信息 |
| 9 | WES→判断 | 判断是否存在异常 |
| 10 | WES→IAC | 成功/失败回告 |

### 异常处理 / Exception Handling

- **校验不通过**: WES下发输送线回退任务 → WES向IAC下发去异常口的搬运任务
- **异常处理完成**: 自动下发入四穿库的Induction

---

## 出库流程 / Outbound Flow

### 整体流程

```
Prime → Pick_Request → WES分配库存 → WCS调度 → 四穿 → 出库口 → 输送带 → AGV/叉车
```

### 指定托盘出库

1. Prime下发托盘号和Footprint Code
2. WES严格按照托盘号分配（忽略质量状态）
3. 如需倒库，先执行倒库再出库
4. 回传pick_confirm

### 质量状态变更出库

- 状态=U：正常回传pick_confirm
- 状态≠U：回传move_error，显示"出库下架中变更质量状态"

---

## 入库/出库切换 / Port Switch

### 切换触发

现场操作员旋转切换按钮 → PLC发送信号 → WCS → WES → IAC

### 切换流程

1. 现场旋转切换按钮
2. PLC发送切换信号给WCS
3. WCS转发给WES
4. WES转发给IAC WCS
5. IAC锁闭切换的作站台
6. 受理/执行的任务继续完成
7. 新建任务终点不定位到该作站台
8. 任务完成后发送完成信号
9. WES切换作站台状态
10. IAC解除锁闭
11. WES→WCS→PLC切换成功
12. PLC指示灯显示可出库

---

## 与Prime接口详情 / Prime Interface

### 入库Induction

| 字段 | 说明 |
|------|------|
| ULID | 容器ID |
| PRTNUM | 物料编码 |
| PRTQTY | 数量 |
| TRANID | 外部单号 |
| 起点排位 | Prime排位 |
| 终点排位 | 目标排位 |

### 出库Pick_Request

| 字段 | 说明 |
|------|------|
| WRKREF | Work Reference |
| 托盘号 | 指定托盘 |
| Batch | 批次 |
| Footprint Code | 托盘规格 |
| 目标排位 | 目标作站台 |

### 回传

| 接口 | 说明 |
|------|------|
| pick_confirm | 正常完成回告 |
| move_error | 异常回告（托盘号+原因） |

---

## 与IAC/AGV接口 / IAC/AGV Interface

### AGV入库

- 入库口容量设置为2（双托盘）
- IAC WCS下发两个容器到缓存位
- 判断作站台状态（空闲/占用）
- 空闲时IAC占用作站台
- AGV托盘放下完成二段任务
- WES外检称重扫码通过后回告IAC释放作站台

### 异常处理

- 校验不通过：WES下发输送线回退任务
- 异常处理完成后自动下发入库Induction

---

## 出入口业务表 / Port Business Table

| 入口 | 入库工作站 | 出库工作站 | 业务 |
|--------|----------|----------|------|
| ASRSRPM_IN01~08 | AGV入库01~08 | AGV异常/出库01~08 | 主线材料供线/退料/CFC2 WIP |
| ASRSRPM_IN09 | 叉车入库01 | 叉车异常/出库01~02 | 仓库材料收货/提货 |
| ASRSRPM_IN10 | 叉车入库02 | 叉车异常/出库01~02 | 仓库材料收货/提货 |

---

## 定时同步 / Scheduled Sync

- **库存同步**: 定时回传WES库存给Prime（可配置时间）
- **状态回传**: 任务状态变更时主动回传
- **差异比对**: 30分钟/次检查库存差异