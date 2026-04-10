# 业务规则 / Business Rules

## 上架策略 / Putaway Strategy

### 优先级 / Priority

1. Same SKU + Same Batch + Same Location Group (同SKU同批次同库位组)
2. Empty Location Group - Deepest Position (空库位组最深层)
3. Exception - Transfer to BCP location (异常 → 异常站)

### 特殊规则 / Special Rules

- **退料 / Returns**: 优先单深位 / Single-deep preferred
- **非退料 / Non-returns**: 优先多深位 / Multi-deep preferred
- **物料类型 / Item Family**: 配置是否允许入四穿库

### 货位锁定 / Location Lock

- 创建虚拟容器时锁闭整个库位组
- 四穿车顶起托盘时自动解锁

---

## 分配策略 / Allocation Strategy

### 出库优先级 / Outbound Priority

1. **Prime指定托盘号**: 严格按照托盘号分配
2. **指定Batch + Footprint Code**: 按MANDTE从小到大
3. **仅指定Footprint Code**: 按MANDTE排序，找最早批次

### 质量状态 / Quality Status

- Status = U: 正常出库，回传pick_confirm
- Status ≠ U: 标记异常，回传move_error

---

## 库存状态 / Inventory Status

### 入库 / Inbound

1. WES接收 → 库存至上架暂存区
2. 四穿到达终点 → 库存移至密集存储区

### 出库 / Outbound

1. WCS顶起托盘 → 库存移至下架暂存区
2. 到达出库工作站前端 → 任务完结
3. 光电感应为空 → 扣减库存

---

## 批次规则 / Batch Rules

- 同一库位组只存放相同SKU相同批次
- 超水位提示不允许入库
- 预留空位用于倒库

### 批次属性 / Batch Attributes

- LOTNUM (批号 / Batch Number)
- FTPCOD (Footprint Code)
- MANDTE (生产日期 / Manufacture Date)
- INVSTS (库存状态 / Inventory Status)
- EXPIRE_DTE (过期日期 / Expiration Date)
- 货主 (Customer: pc04-pc09)

---

## 任务优先级 / Task Priority

- 数值越低，优先级越高
- 新建状态可修改优先级
- 范围: 1-50

### 优先级升级 / Priority Upgrade (IAC)

- Range: 1-50
- Start: 50, End: 1
- Step: 每10分钟降2级 / Every 10 min: -2

---

## 库存质量状态变更 / Quality Status Change

### 入库时

- 上架任务执行完成后，库存状态变为最新的质量状态

### 出库时

- 质量状态=U：正常回传pick_confirm
- 质量状态≠U：标记异常，回传move_error

---

## 虚拟容器 / Virtual Container

### 创建流程

1. 运维选择空闲货位
2. WES搜索货位 → 创建虚拟托盘 ASRS-virtual-0001
3. 系统锁闭整个库位组
4. 现场放空托盘到指定货位
5. 四穿车顶起托盘时自动解锁

### 命名规则

- 前缀：ASRS-virtual-
- 后缀：数值依次递增

### 锁定逻辑

- 上架/下架不再推荐该库位组
- 锁闭前有新建下架任务 → 允许创建虚拟托盘
- 托盘离开库位组后恢复推荐