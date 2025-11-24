# PDXC-SRS



## 1、**PDX1D/PDX1DV**

> 开环控制：
>
> Position: target and actual position, range -10.00000 to +10.00000 mm,
>
> Velocity: 1 ~10 mm/s,
>
> Jog Step: distance for each move action,0.0001 to 10.00000mm



> 闭环控制：
>
> Position: target and actual position, range -10.00000 to +10.00000 mm,
>
> Velocity: 1~10 mm/s,
>
> Jog Step: distance for each move action, 0.0001 to 10.00000 mm



> - Analog In 范围：-10V 到 10V → 代表 -10.00 到 10.00 mm（单位：mm）
>
> - AiOffsetMin = -10
>
> - AiOffsetMax = 10
>
> - 单位：mm → AiOffsetMultiple = 1（不需要转换）



## 2、**PDXG1**

> 开环控制：
>
>  Position: target and actual position, range -8.00000 ==degrees== to +8.00000 ==degrees==
>
> Velocity: 1~10 ==degree/s==,
>
> Jog Step: distance for each move action, 0.0004 to 8.000000 ==degrees==



> 闭环控制：
>
> Position: target and actual position, range -8.00000 ==degrees== to +8.00000 ==degrees==
>
> Velocity:  1~10 ==degree/s==,
>
> Jog Step: distance for each move action, 0.0004 to 8.000000 ==degrees==



> ### PDXG1
>
> - Analog In 范围：-10V 到 10V → 代表 -8.00 到 8.00 degrees（单位：degrees）
>
> - AiOffsetMin = -8
>
> - AiOffsetMax = 8
>
> - 单位：degrees → AiOffsetMultiple = 0.01（需要转换）





> **模拟输入（Analog In）：**
>
> 用户可设置模拟输入信号的增益（Gain）和偏移量（Offset）来表征平台的位置。增益范围从 0.1 到 1，因此目标位置 = 输入电压 × 增益 + 偏移量。默认情况下，模拟输入信号的范围为 -10V 到 10V，代表目标位置范围 ==-8.00 到 8.00 度==，此时增益为 ==1.25x==，偏移量为 0。
>
> **固定步长（Fixed Step）：**
> 在此操作模式下，触发边沿每次将触发平台按指定步长值移动一段距离。
>
> **固定位置（Fixed Positions）：**
> 在此操作模式下，位置1（Pos1）将在其选定的触发边沿执行，位置2（Pos2）同样会在其选定的触发边沿执行。这两个步骤将以交替方式执行。
>
> ------
>
> **关键术语说明：**
>
> - **触发边沿（Trigger Edge）**：指信号上升沿或下降沿的触发条件
> - **交错方式（Staggered way）**：指两个位置交替执行，形成交替运动模式
> - **增益与偏移**：用于校准电压信号与物理位置的对应关系