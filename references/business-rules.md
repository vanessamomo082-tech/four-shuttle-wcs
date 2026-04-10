# 业务规则 / Business Rules

## 上架策略

1. 同SKU同批次 → 同一库位组
2. 空位组最深层
3. 异常→暂存排位

## 分配策略

- 按托盘号分配（精确）
- 按Batch+Footprint分配（按日期）
- 按Footprint分配（最早批次）

## 质量状态

- U：正常出库
- 非U：异常回传move_error

## 批次属性

- LOTNUM：批号
- FTPCOD：托盘规格
- MANDTE：生产日期
- INVSTS：库存状态
- EXPIRE_DTE：过期日期

## 优先级

- 范围：1-50，数值越低越高
- 升级：每10分钟降2级