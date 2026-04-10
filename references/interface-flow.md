# 接口流程 / Interface Flow

## 入库流程

```
Prime → IAC → RCS → 缓存位 → 四穿入库口 → WES → WCS → 四穿 → 上架
```

详细：
1. Prime→IAC：下发Induction
2. IAC→RCS：一段任务到缓存位
3. RCS→IAC：搬运完成
4. WES→IAC：作站台状态
5. IAC→RCS：放到入库口
6. PLC→WCS：光电感应有占位
7. WCS→WES：设备识别
8. WES判断：成功/失败回告

## 出库流程

```
Prime → Pick_Request → WES → WCS → 四穿 → 出库口 → 输送带
```

## 切换流程

1. 现场旋转切换按钮
2. PLC→WCS→WES→IAC
3. 锁闭作站台
4. 任务完成后切换

## 接口详情

### 入库Induction

| 字段 | 说明 |
|------|------|
| ULID | 容器ID |
| PRTNUM | 物料编码 |
| PRTQTY | 数量 |

### 出库Pick_Request

| 字段 | 说明 |
|------|------|
| 托盘号 | 指定托盘 |
| Batch | 批次 |
| Footprint | 规格 |

### 回传

- pick_confirm：正常完成
- move_error：异常（托盘号+原因）

## 出入口

| 入口 | 入库工作站 | 出库工作站 |
|------|-----------|-----------|
| 1-8 | AGV01-08 | AGV异常/出库01-08 |
| 9 | 叉车01 | 叉车异常/出库01-02 |
| 10 | 叉车02 | 叉车异常/出库01-02 |