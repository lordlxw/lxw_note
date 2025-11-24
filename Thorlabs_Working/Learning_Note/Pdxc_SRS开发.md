我来仔细分析这个软件需求，看看需要在 `PDXCParas.cs` 和 `PDX1ViewModel.cs` 这两个文件中做哪些改动。


[1 tool called]

根据软件需求分析，我发现需要修改的主要参数范围：

## 需求分析总结：

### PDX1D/PDX1DV 平台：
- **位置范围**：-10.00000 到 +10.00000 mm
- **速度范围**：1 ~ 10 mm/s  
- **步进距离**：0.0001 到 10.00000 mm

### PDXG1 平台：
- **位置范围**：-8.00000 到 +8.00000 度
- **速度范围**：1 ~ 10 度/s (开环), 1 度/s (闭环)
- **步进距离**：0.0004 到 8.000000 度

## 需要修改的代码位置：


[1 tool called]

### 1. PDXCParas.cs 中需要修改的地方：

**AoOffsetMin 和 AoOffsetMax 属性**（第169-216行）：
- 当前 PDX1D/PDX1DV 和 PDXG1 都返回 -10 到 10
- 需要修改为：
  - PDX1D/PDX1DV: -10 到 10 (保持不变)
  - PDXG1: -8 到 8


[1 tool called]

### 2. PDX1ViewModel.cs 中需要修改的地方：

**位置限制范围**（第153-202行）：
- `PositionLimitRangeMin` 和 `PositionLimitRangeMax`
- 当前 PDX1D/PDX1DV 和 PDXG1 都返回 -10 到 10
- 需要修改为：
  - PDX1D/PDX1DV: -10 到 10 (保持不变)
  - PDXG1: -8 到 8

**速度范围**（第238-287行）：
- `SetSpeedMin` 和 `SetSpeedMax`
- 当前 PDX1D/PDX1DV: 2 到 20 mm/s
- 当前 PDXG1: 2 到 20 度/s
- 需要修改为：
  - PDX1D/PDX1DV: 1 到 10 mm/s
  - PDXG1: 1 到 10 度/s (开环), 1 度/s (闭环)

**步进距离范围**（第303-354行）：
- `JogStepMin` 和 `JogStepMax`
- 当前 PDX1D/PDX1DV: 0.00001 到 10
- 当前 PDXG1: 0.00001 到 10
- 需要修改为：
  - PDX1D/PDX1DV: 0.0001 到 10.00000
  - PDXG1: 0.0004 到 8.000000

**触发模式参数范围**（第422-468行）：
- `TriggerFSSStepSizeMin` 和 `TriggerFSSStepSizeMax`
- 当前 PDX1D/PDX1DV: -10 到 10
- 当前 PDXG1: -10 到 10
- 需要修改为：
  - PDX1D/PDX1DV: -10 到 10 (保持不变)
  - PDXG1: -8 到 8

**模拟输入偏移范围**（第602-649行）：
- `AiOffsetMin` 和 `AiOffsetMax`
- 当前 PDX1D/PDX1DV: -10 到 10
- 当前 PDXG1: -10 到 10
- 需要修改为：
  - PDX1D/PDX1DV: -10 到 10 (保持不变)
  - PDXG1: -8 到 8


[1 tool called]

## 总结需要修改的具体位置：

### PDXCParas.cs 修改点：
1. **第169-216行**：`AoOffsetMin` 和 `AoOffsetMax` 属性
   - 为 PDXG1 添加 -8 到 8 的范围限制

### PDX1ViewModel.cs 修改点：
1. **第153-202行**：`PositionLimitRangeMin` 和 `PositionLimitRangeMax`
   - 为 PDXG1 添加 -8 到 8 的位置限制

2. **第238-287行**：`SetSpeedMin` 和 `SetSpeedMax`
   - PDX1D/PDX1DV: 改为 1 到 10
   - PDXG1: 改为 1 到 10

3. **第303-354行**：`JogStepMin` 和 `JogStepMax`
   - PDX1D/PDX1DV: 改为 0.0001 到 10.00000
   - PDXG1: 改为 0.0004 到 8.000000

4. **第422-468行**：`TriggerFSSStepSizeMin` 和 `TriggerFSSStepSizeMax`
   - 为 PDXG1 添加 -8 到 8 的范围限制

5. **第602-649行**：`AiOffsetMin` 和 `AiOffsetMax`
   - 为 PDXG1 添加 -8 到 8 的范围限制

这些修改将确保软件参数范围与新的需求规格保持一致，特别是针对 PDXG1 平台的位置范围从 ±10 度改为 ±8 度，以及速度范围的调整。