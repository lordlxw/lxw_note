> 开发开始时间：2025/09/15







## 功能点修复

> 1、改log，加日期，开始时间，结束时间，ElapsedTime
>
> Detail information should be shown, such as record date, sample period, sample length.
>
> 图表上加上时间，即纵轴。
>
> 
>
> ![image-20250915140719927](C:\Users\root\AppData\Roaming\Typora\typora-user-images\image-20250915140719927.png)
>
> 难点：需要记录停止的时间
>
> 现状：头部每次重写，数据是叠加的。
>
> 停止其实分几种：
> 1、自然停止，当到达边界时自动结束
> 2、手动停止，用户点击停止键
> 3、异常退出或手动退出的停止



> 2、改样式，左边图，右边配置（就是csv文件）



> 3、Trigger 和 Analog
>
> 启用 TRIGGERIN 后，温度控制项无法通过点击更改，其颜色也会更改为表示它在与正常状态不同的状态下工作，它应该根据 TRIGGERIN 信号（从 SCPI 命令读取）更新其状态。
>
> 启用 ANALOGINPUT 后，无法设置 Target 功能，其颜色也更改为表示它在与正常状态不同的状态下工作，它应该根据输入模拟信号更新显示的数字（从 SCPI 命令获取）。



> 核心：
>
> Update 2025.9.2
>
> - Software logging time is not accurate, please add time record.
> - Add cycle setup function as the firmware have added cycle function.
> - IP地址加密？





## 会议

> 1、CircleMode逻辑改变
>
> 实现从软件变成硬件
>
> 2、报错时候能响应命令吗？（disable）？
>
> 不用，报错只要显示一下。
>
> 3、界面上日志能加载？（似乎从硬件里读）？





## 核心串口通信命令

> 1、SENsor:TEMPerature:ACTualtemp?
>
> 获取实时温度



# 问题

> 1、When the Heating/Cooling selection is changed, the minimal value of period should longer than 5 minutes.
>
> 好像已经实现了。切换大于5min



> 2、定时设置硬件的问题：
>
> - 多次等待会累积时间误差
>
> - 每次 Monitor.Wait 都可能有多毫秒的偏差
>
> Log 和 Sequence 任务怎么同步呢？



> 3、新的错误位定义，好像是针对压缩机的？
>
> ==compressor error==
>
> bit0: Software Over Current        (软件过流)
> bit1: Over Voltage Protection     (过压保护)
> bit2: Under Voltage Protection    (欠压保护)
> bit3: Phase Loss Protection        (缺相保护)
> bit4: Speed Failure Protection     (速度故障保护)
> bit5: Hardware Over Current       (硬件过流)
> bit6: Phase Current Anomaly        (相电流异常)
> bit7: No Com                      (无通信)



> 4、加密的问题，
>
> If the upper computer actively initiates paging or the refresh button is pressed, the encryption status will be displayed based on the state returned by the device, and modifying the encryption status will not be supported.
>
> 加密的状态位是由硬件给的？可以获取，修复了



> 5、删去的时候会出现°C消失的情况。







## 核心C++通信

> uart库是负责串口通信的。
>
> uart_library 提供的核心函数：
>
> 1. ✅ fnUART_LIBRARY_list - 设备发现
> 2. ✅ fnUART_LIBRARY_open - 设备连接
> 3. ✅ fnUART_LIBRARY_write - 数据发送
> 4. ✅ fnUART_LIBRARY_read - 数据接收
> 5. ✅ fnUART_LIBRARY_close - 设备关闭
>
> 这些函数提供了完整的USB/串口设备管理功能，支持设备发现、连接、数据传输和断开连接。在LK500项目中，这些函数被封装在 UsbCommandCommunicator 类中，用于实现USB设备的SCPI命令通信。





> 新增的循环相关的接口
>
> 



### 1、CONTrol:CYCle:CycleInfo?	

> 方法名——Get_Cycle_Info
>
> - 查询Cycle的完整信息
>
> - ### 返回数据格式
>
>   返回9个用逗号分隔的数值，按顺序包含：
>
>   例子：0,1000,500,56,28,3,2,23.2,22.21
>
>   state - Cycle状态
>
>   - 0 = IDLE（空闲）
>
>   - 1 = RUNNING（运行中）
>
>   - 2 = PAUSED（暂停）
>
>   
>
>   
>
>   total time - 总时间（分钟）
>
>   - 整个Cycle的预计总执行时间
>
>   
>
>   
>
>   passed time - 已用时间（分钟）
>
>   - 已经执行的时间
>
>   
>
>   stepnum - 步骤总数
>
>   - 每个周期包含的步骤数量
>
>   
>
>   current step - 当前步骤
>
>   - 正在执行的步骤编号
>
>   
>
>   cyclenum - 周期总数
>
>   - 总共要执行的周期数
>
>   
>
>   current cycle - 当前周期
>
>   - 正在执行的周期编号
>
>   
>
>   setup temp - 设置温度
>
>   - 当前步骤的目标温度
>
>   - 精度：0.1°C
>
>   
>
>   actual temp - 实际温度
>
>   - 设备当前的实时温度
>
>   - 精度：0.01°C



### 2、CONTrol:CYCle:CYCNum  [cyclenum]

> 方法名——Set_Cycle_CYCNum
>
> CONTrol:CYCle:CYCNum <cyclenum> - 设置Cycle周期数量
>
> ### 参数说明
>
> - cyclenum：周期重复次数
>
> - 类型：uint（无符号整数）
>
> - 范围：1 到 100
>
> ### 功能
>
> - 设置Cycle要重复执行的周期数量
>
> - 限制：Cycle运行期间无效（只能在停止状态下设置）
>
> ```scpi
> CONTrol:CYCle:CYCNum 10
> //设置Cycle重复执行10个周期
> ```
>
> ### 使用场景
>
> - 在开始Cycle之前，先设置要执行多少个周期
>
> - 比如设置10个周期，每个周期有5个步骤，那么总共会执行50个步骤
>
> - 一旦Cycle开始运行，就不能再修改这个值
>
> ### 注意事项
>
> - 必须在Cycle停止状态下才能设置
>
> - 范围限制在1-100之间
>
> - 设置后会影响整个Cycle的执行次数





### 3、CONTrol:CYCle:CYCNum?

> 方法名——Get_Cycle_CYCNum
>
> 查询循环次数





### *4、CONTrol:CYCle:HALt [state]

> 设置停止状态
>
> ```scpi
> CONTrol:CYCle:HALt 1
> ```
>
> 
>
> ### 参数说明
>
> - state：暂停状态
>
> - 类型：uint（无符号整数）
>
> - 范围：0 或 1
>
> ### 参数值含义
>
> - 0 = disabled（禁用暂停）
>
> - 1 = enabled（启用暂停）
>
> ### 功能
>
> - 设置Cycle的暂停状态
>
> - 限制：Cycle运行期间无效（只能在停止状态下设置）



### 5、CONTrol:CYCle:HALt?

> 查询停止状态位



### 6、CONTrol:CYCle:State?

> 查询循环状态
>
> 0:disabled 1:enabled



### 7、CONTrol:CYCle:STEPInfo [ramptime], [holdtime], [holdtemp], [RunMode]

> ```scpi
> CONTrol:CYCle:STEPInfo 1000, 1000, 24.5, 1
> 升温时间：1000分钟
> 保持时间：1000分钟
> 目标温度：24.5°C
> 运行模式：1（加热模式）
> ```
>
> #### 1. ramptime（升温时间）
>
> - 范围：0 ~ 6000 分钟
>
> - 含义：从当前温度升/降到目标温度所需的时间
>
> #### 2. holdtime（保持时间）
>
> - 范围：0 ~ 6000 分钟
>
> - 含义：在目标温度下保持的时间
>
> #### 3. holdtemp（保持温度）
>
> - 范围：-5 ~ 45°C
>
> - 含义：步骤的目标温度
>
> #### 4. RunMode（运行模式）
>
> - 范围：0 ~ 1
>
> - 含义：工作模式
>
> - 0 = Cooling（冷却模式）
>
> - 1 = Heating（加热模式）
>
> ### 功能
>
> - 设置Cycle中单个步骤的详细参数
>
> - 限制：Cycle运行期间无效（只能在停止状态下设置）
>
> ### 注意事项
>
> - 必须在Cycle停止状态下才能设置
>
> - 时间单位是分钟
>
> - 温度单位是摄氏度
>
> - 设置后会影响Cycle执行时的具体行为



### 8、CONTrol:CYCle:STEPInfo?

> 返回4个用逗号分隔的数值，按顺序包含：
>
> ramptime - 升温时间（分钟）
>
> - 范围：0 ~ 6000 分钟
>
> - 从当前温度升/降到目标温度所需的时间
>
> 
>
> holdtime - 保持时间（分钟）
>
> - 范围：0 ~ 6000 分钟
>
> - 在目标温度下保持的时间
>
> 
>
> holdtemp - 保持温度（°C）
>
> - 范围：-5 ~ 45°C
>
> - 步骤的目标温度
>
> 
>
> RunMode - 运行模式
>
> - 范围：0 ~ 1
>
> - 0 = Cooling（冷却模式）
>
> - 1 = Heating（加热模式）





### 9、CONTrol:CYCle:STEPNum  [stepnum]

> ### 命令名称
>
> CONTrol:CYCle:STEPNum <stepnum> - 设置步骤数量
>
> ### 参数说明
>
> - stepnum：每个周期中的步骤数量
>
> - 类型：uint（无符号整数）
>
> - 范围：1 到 100
>
> ### 功能
>
> - 设置每个周期包含多少个步骤
>
> - 限制：Cycle运行期间无效（只能在停止状态下设置）
>
> ### 示例text>> CONTrol:CYCle:STEPNum 30
>
> - 设置每个周期包含30个步骤
>
> ### 使用场景
>
> - 在开始Cycle之前，设置每个周期的步骤总数
>
> - 比如设置30个步骤，那么每个周期会执行30个步骤
>
> - 需要配合 CONTrol:CYCle:STEPInfo 命令来设置每个步骤的具体参数
>
> ### 注意事项
>
> - 必须在Cycle停止状态下才能设置
>
> - 范围限制在1-100之间
>
> - 设置后会影响整个Cycle的结构
>
> - 需要为每个步骤单独配置参数（温度、时间、模式等）
>
> ### 相关命令
>
> - 通常配合 CONTrol:CYCle:STEPNum? 查询当前设置的步骤数量
>
> - 需要配合 CONTrol:CYCle:STEPInfo 设置每个步骤的详细参数



### 10、CONTrol:CYCle:STEPNum?

> 命令查询的是总步骤数量
>
> range from 1 to 100



### *11、CONTrol:CYCle:STEPSel  [stepsel]  

>  设置选中的步骤编号
>
> ### 参数说明
>
> - stepsel：可以编辑的步骤编号
>
> - 类型：uint（无符号整数）
>
> - 范围：1 到 100
>
> ### 功能
>
> - 选择要编辑或查看的步骤
>
> - 限制：Cycle运行期间无效（只能在停止状态下设置）
>
> ### 示例
>
> ```scpi
> CONTrol:CYCle:STEPSel 30
> ```
>
> - 选择第30个步骤进行编辑
>
> ### 使用场景
>
> - 在设置步骤参数之前，先选择要编辑哪个步骤
>
> - 选择步骤后，可以使用 CONTrol:CYCle:STEPInfo 设置该步骤的参数
>
> - 可以使用 CONTrol:CYCle:STEPInfo? 查询该步骤的当前参数
>
> ### 工作流程
>
> 1. 先设置总步骤数量：CONTrol:CYCle:STEPNum 30
>
> 1. 选择要编辑的步骤：CONTrol:CYCle:STEPSel 5
>
> 1. 设置该步骤参数：CONTrol:CYCle:STEPInfo 100, 100, 25.0, 1
>
> 1. 重复步骤2-3，设置其他步骤
>
> ### 注意事项
>
> - 必须在Cycle停止状态下才能设置
>
> - 范围限制在1-100之间
>
> - 选择的步骤编号不能超过总步骤数量
>
> - 设置后会影响后续的 STEPInfo 命令操作



### 12、CONTrol:CYCle:STEPSel?

> 查询选中的Step Number
>
> range:1-100



### 13、CONTrol:RUNMode [state]

> 设置运行模式。
>
> 0:CONSTANT 1:CYCLE



### 14、CONTrol:RUNMode?

> 查询运行模式
>
> 0:CONSTANT T 1:PAUSE





### 执行顺序

> ## 完整的配置流程（带验证）
>
> ### 第1步：
>
> ### 检查当前状态 
>
> ### CONTrol:CYCle:CycleInfo?    // 查询当前Cycle状态  // 确保state=0 (IDLE)，才能进行配置
>
> 
>
> ### 第2步：设置周期数量 
>
> ### CONTrol:CYCle:CYCNum 5     // 设置5个周期
>
> ### CONTrol:CYCle:CYCNum?      // 验证设置是否成功
>
> 
>
> ### 第3步：设置步骤数量 
>
> ### CONTrol:CYCle:STEPNum 10    // 设置10个步骤
>
> ###  CONTrol:CYCle:STEPNum?     // 验证设置是否成功
>
> 
>
> ### 第4步：配置每个步骤
>
> ### CONTrol:CYCle:STEPSel 1     // 选择第1步
>
> ### CONTrol:CYCle:STEPSel?     // 验证选择是否成功
>
> ### CONTrol:CYCle:STEPInfo 100,100,25.0,1 // 配置第1步
>
> ### CONTrol:CYCle:STEPInfo?     // 验证配置是否成功
>
> ---
>
> ==以此类推！！！==
>
> ### CONTrol:CYCle:STEPSel 2    // 选择第2步11. CONTrol:CYCle:STEPSel?     // 验证选择是否成功12. CONTrol:CYCle:STEPInfo 200,150,30.0,0 // 配置第2步13. CONTrol:CYCle:STEPInfo?    // 验证配置是否成功... (重复配置所有10个步骤)
>
> 
>
> ### 第5步：启用暂停功能（可选）
>
> ### CONTrol:CYCle:HALt 1      // 启用暂停功能
>
> ### CONTrol:CYCle:HALt?      // 验证设置是否成功
>
> 
>
> ### 第6步：启动
>
> ### CONTrol:RUNMode 1       // 启动CYCLE模式
>
> ### CONTrol:RUNMode?        // 验证启动是否成功
>
> 
>
> ### 第7步：监控状态
>
> ### CONTrol:CYCle:CycleInfo?    // 持续查询执行状态
>
> ### CONTrol:CYCle:State?      // 查询Cycle状态
>
> ## 关键点
>
> - 每个设置命令后都要查询验证
> - 必须先检查Cycle状态是否为IDLE
> - 设置步骤前必须先选择步骤
> - 所有配置必须在IDLE状态下完成
>
> ## 错误处理
>
> 如果任何查询返回的值与预期不符，需要：
>
> - 停止配置流程
>
> - 报告错误
>
> - 等待设备回到IDLE状态后重试





### 问题

> 1、设置完参数后如何启动？
>
> 2、启动完如何暂停和停止？







# 重构dll

> 1、Open_Ethernet_External —— 7个参数改成6个，因为方法里有返回
>
> 删除第7个参数 cancelEvent，因为新版本的 ethernet_library 内部已经处理了取消功能。







## 初始化的时候我需要获取的数据

> 1、当前执行状态
>
> 2、总步骤数，总循环数，每个步骤的详情设置（显示在datagrid上）
>
> 3、执行到哪一步的状态







### 注意点：

> 1、循环模式下才有暂停
>
> 2、Ethernet加端口号。





# SCPI命令集合

## MainContentViewModel 中使用的 SCPI 指令对照表

| C# 方法                      | SCPI 指令                            | 参数类型                   | 说明               |
| :--------------------------- | :----------------------------------- | :------------------------- | :----------------- |
| 控制和状态                   |                                      |                            |                    |
| GetIDN()                     | *IDN?                                | -                          | 获取设备标识       |
| SetIsTempControlOn(value)    | CONTrol:OUTPut:STATe <value>         | int (0/1)                  | 设置温度控制开关   |
| GetIsTempControlOn()         | CONTrol:OUTPut:STATe?                | -                          | 获取温度控制状态   |
| SetRunMode(value)            | CONTrol:RUNMode <value>              | int (0=CONSTANT, 1=CYCLE)  | 设置运行模式       |
| GetRunMode()                 | CONTrol:RUNMode?                     | -                          | 获取运行模式       |
| SetWorkMode(value)           | CONTrol:OUTPut:WORKmode <value>      | int (0=Cooling, 1=Heating) | 设置工作模式       |
| GetWorkMode()                | CONTrol:OUTPut:WORKmode?             | -                          | 获取工作模式       |
| SetTriggerInState(value)     | CONTrol:TRIGger:ENabled <value>      | int (0/1)                  | 设置触发输入使能   |
| GetTriggerInState()          | CONTrol:TRIGger:ENabled?             | -                          | 获取触发输入状态   |
| SetAnalogInState(value)      | CONTrol:ANALoginput:ENabled <value>  | int (0/1)                  | 设置模拟输入使能   |
| GetAnalogInState()           | CONTrol:ANALoginput:ENabled?         | -                          | 获取模拟输入状态   |
| 驱动和传感器                 |                                      |                            |                    |
| SetFanDriverMode(value)      | CONTrol:DRViver:FANDRiver <value>    | int                        | 设置风扇驱动模式   |
| GetFanDriverMode()           | CONTrol:DRViver:FANDRiver?           | -                          | 获取风扇驱动模式   |
| SetPumpDriverLevel(value)    | CONTrol:DRViver:PUMPDRiver <value>   | int                        | 设置泵驱动级别     |
| GetPumpDriverLevel()         | CONTrol:DRViver:PUMPDRiver?          | -                          | 获取泵驱动级别     |
| SetSensorSource(value)       | SENsor:TEMPerature:SOURce <value>    | int                        | 设置传感器源       |
| GetSensorSource()            | SENsor:TEMPerature:SOURce?           | -                          | 获取传感器源       |
| 温度相关                     |                                      |                            |                    |
| SetTemperatureUnit(value)    | SENsor:TEMPerature:UNIT <value>      | int                        | 设置温度单位       |
| GetTemperatureUnit()         | SENsor:TEMPerature:UNIT?             | -                          | 获取温度单位       |
| SetTargetTemperature(value)  | CONTrol:TEMPerature:SPOint <value>   | float                      | 设置目标温度       |
| GetTargetTemperature()       | CONTrol:TEMPerature:SPOint?          | -                          | 获取目标温度       |
| SetWindowsTemperature(value) | CONTrol:TEMPerature:ERRWin <value>   | float                      | 设置温度窗口       |
| GetWindowsTemperature()      | CONTrol:TEMPerature:ERRWin?          | -                          | 获取温度窗口       |
| GetActualTemperature()       | SENsor:TEMPerature:ACTualtemp?       | -                          | 获取实际温度       |
| GetIncomeTemperature()       | SENsor:TEMPerature:INcome?           | -                          | 获取入口温度       |
| GetOutcomeTemperature()      | SENsor:TEMPerature:OUTcome?          | -                          | 获取出口温度       |
| GetAmbientTemperature()      | SENsor:AMBient:TEMPerature?          | -                          | 获取环境温度       |
| GetAmbientHumidity()         | SENsor:AMBient:HUMidity?             | -                          | 获取环境湿度       |
| PID 参数 - 加热              |                                      |                            |                    |
| SetHeatKp(value)             | CONTrol:PID:HEATing:KP <value>       | float                      | 设置加热 PID Kp    |
| GetHeatKp()                  | CONTrol:PID:HEATing:KP?              | -                          | 获取加热 PID Kp    |
| SetHeatTi(value)             | CONTrol:PID:HEATing:TI <value>       | float                      | 设置加热 PID Ti    |
| GetHeatTi()                  | CONTrol:PID:HEATing:TI?              | -                          | 获取加热 PID Ti    |
| SetHeatTd(value)             | CONTrol:PID:HEATing:TD <value>       | float                      | 设置加热 PID Td    |
| GetHeatTd()                  | CONTrol:PID:HEATing:TD?              | -                          | 获取加热 PID Td    |
| SetHeatPeriod(value)         | CONTrol:PID:HEATing:PERiod <value>   | float                      | 设置加热 PID 周期  |
| GetHeatPeriod()              | CONTrol:PID:HEATing:PERiod?          | -                          | 获取加热 PID 周期  |
| PID 参数 - 冷却              |                                      |                            |                    |
| SetCoolKp(value)             | CONTrol:PID:COOLing:KP <value>       | float                      | 设置冷却 PID Kp    |
| GetCoolKp()                  | CONTrol:PID:COOLing:KP?              | -                          | 获取冷却 PID Kp    |
| SetCoolTi(value)             | CONTrol:PID:COOLing:TI <value>       | float                      | 设置冷却 PID Ti    |
| GetCoolTi()                  | CONTrol:PID:COOLing:TI?              | -                          | 获取冷却 PID Ti    |
| SetCoolTd(value)             | CONTrol:PID:COOLing:TD <value>       | float                      | 设置冷却 PID Td    |
| GetCoolTd()                  | CONTrol:PID:COOLing:TD?              | -                          | 获取冷却 PID Td    |
| SetCoolPeriod(value)         | CONTrol:PID:COOLing:PERiod <value>   | float                      | 设置冷却 PID 周期  |
| GetCoolPeriod()              | CONTrol:PID:COOLing:PERiod?          | -                          | 获取冷却 PID 周期  |
| 传感器参数                   |                                      |                            |                    |
| SetNTC_Type(value)           | SENsor:TEMPerature:NTCType <value>   | int                        | 设置 NTC 类型      |
| GetNTC_Type()                | SENsor:TEMPerature:NTCType?          | -                          | 获取 NTC 类型      |
| SetCoefficientType(value)    | SENsor:TEMPerature:COEFFType <value> | int                        | 设置系数类型       |
| GetCoefficientType()         | SENsor:TEMPerature:COEFFType?        | -                          | 获取系数类型       |
| SetBetaValue(value)          | SENsor:TEMPerature:BETA <value>      | int                        | 设置 Beta 值       |
| GetBetaValue()               | SENsor:TEMPerature:BETA?             | -                          | 获取 Beta 值       |
| SetR25BetaValue(value)       | SENsor:TEMPerature:R25 <value>       | int                        | 设置 R25 值        |
| GetR25BetaValue()            | SENsor:TEMPerature:R25?              | -                          | 获取 R25 值        |
| SetSensorAValue(value)       | SENsor:TEMPerature:Apara <value>     | float                      | 设置传感器 A 参数  |
| GetSensorAValue()            | SENsor:TEMPerature:Apara?            | -                          | 获取传感器 A 参数  |
| SetSensorBValue(value)       | SENsor:TEMPerature:Bpara <value>     | float                      | 设置传感器 B 参数  |
| GetSensorBValue()            | SENsor:TEMPerature:Bpara?            | -                          | 获取传感器 B 参数  |
| SetSensorCValue(value)       | SENsor:TEMPerature:Cpara <value>     | float                      | 设置传感器 C 参数  |
| GetSensorCValue()            | SENsor:TEMPerature:Cpara?            | -                          | 获取传感器 C 参数  |
| 其他传感器                   |                                      |                            |                    |
| GetPressure()                | SENsor:PREssure?                     | -                          | 获取压力           |
| GetFlowRate()                | SENsor:FLOw?                         | -                          | 获取流速           |
| 状态和错误                   |                                      |                            |                    |
| GetErrorState()              | SYSTem:STAte:ERRor?                  | -                          | 获取错误状态       |
| GetWarningState()            | SYSTem:STAte:WARNing?                | -                          | 获取警告状态       |
| GetCompErrorState()          | SYSTem:STAte:CompERRor?              | -                          | 获取压缩机错误状态 |
| 显示                         |                                      |                            |                    |
| SetDarkMode(value)           | DISPlay:DARKmode <value>             | int (0/1)                  | 设置暗模式         |
| GetIsDarkMode()              | DISPlay:DARKmode?                    | -                          | 获取暗模式状态     |
| 系统                         |                                      |                            |                    |
| ResetAllParameters()         | SYSTem:LoadFactory                   | -                          | 恢复出厂设置       |

说明：

- ? 结尾的指令是查询指令（Get）

- 不带 ? 的指令是设置指令（Set）

- 数值参数在指令后以空格分隔

- 所有指令以 \n 结尾



# 2025.09.30问题

> 1、回车触发ReqAdd方法，但同时IsDefault=true，也会触发confirmCommand方法。
>
> 2、重构一下EthernetDeviceDetail，用两个类来封装。一个加，一个是真实的类。





> private Task RefreshEthernetDevicesDetailList()
> {
>     return Task.Run(() =>
>     {
>         _ethernetDeviceDetails.Clear();
>         RefreshConnectedEthernetDeviceInfoList();——==GetAvailableEthernetDevices==
>         RefreshSavedEthernetDeviceInfoList();——==GetSavedEthernetDevice==
>         RefreshCustomAddedEthernetDeviceInfoList();——==GetCertainEthernetDevice==
>     });
> }
>
> GetCertainEthernetDevice——GetEthernetDeviceInfoByIP —— 根据IP找设备
>
> GetSavedEthernetDevice——
