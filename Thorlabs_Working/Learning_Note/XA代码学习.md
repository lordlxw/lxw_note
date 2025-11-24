# XA代码学习

> ## 快速理解路径
>
> 1. 看 SystemManager
>
>   └─> 理解：系统如何启动、如何发现设备、如何创建设备
>
> 2. 看 AptModule
>
>   └─> 理解：消息如何接收、如何路由、如何分发
>
> 3. 看 AptDevice
>
>   └─> 理解：设备如何初始化、如何管理生命周期
>
> 4. 看 DeviceFeature
>
>   └─> 理解：功能如何注册、如何处理消息
>
> 5. 看具体设备（如 Kna101）
>
>   └─> 理解：如何继承基类、如何实现特定功能
>
> ## 总结
>
> 核心框架只有这几个类：
>
> - SystemManager - 系统管理
>
> - AptModule - 消息处理核心
>
> - AptDevice - 设备基类
>
> - DeviceFeature - 功能基类
>
> - AptProcessorKey - 消息路由





## SystemManager

> ### 设备发现流程
>
> #### 1. 系统启动时初始化发现服务
>
> *// systemmanager.cpp - Startup()*
>
> _iDeviceDiscovery->Initialize(piDeviceNotificationsHandler);
>
> _iDeviceDiscovery->Start(); *// 启动发现服务*
>
> #### 2. 发现流程（三层架构）
>
> SystemManager
>
>   ↓ 调用
>
> DeviceDiscovery (发现管理器)
>
>   ↓ 遍历传输层
>
> DiscoveredTransport (USB/串口等)
>
>   ↓ 遍历数据流
>
> DiscoveredDataStream (具体连接)
>
>   ↓ 调用
>
> AptDeviceDiscovery (APT协议发现)
>
> #### 3. 核心发现逻辑（AptDeviceDiscovery）
>
> void AptDeviceDiscovery::DiscoverDevices(...)
>
> {
>
>   *// 步骤1: 创建临时发现设备*
>
>   iAptDevice = _iAptDeviceFactory->CreateDiscoveryDevice(...);
>
>   
>
>   *// 步骤2: 获取硬件信息功能*
>
>   pIHardwareInfoDeviceFeature = GetHardwareInfoDeviceFeature();
>
>   
>
>   *// 步骤3: 探测两个控制器地址*
>
>   std::vector<std::byte> addressesToProbe{
>
> ​    0x50, *// USB Controller*
>
> ​    0x11  *// Motherboard Controller*
>
>   };
>
>   
>
>   *// 步骤4: 对每个地址发送查询*
>
>   for (auto address : addressesToProbe) {
>
> ​    ProbeAptAddress(address, ...); *// 发送硬件信息查询*
>
>   }
>
> }
>
> #### 4. 地址探测（ProbeAptAddress）
>
> bool ProbeAptAddress(...)
>
> {
>
>   *// 1. 设置远程地址*
>
>   pISetRemoteAddress->SetRemoteAddress(aptAddress);
>
>   
>
>   *// 2. 发送硬件信息查询命令（等待2秒响应）*
>
>   pIHardwareInfoDeviceFeature->GetHardwareInfo(
>
> ​    &hardwareInfo, 
>
> ​    std::chrono::milliseconds(2000)
>
>   );
>
>   
>
>  
>
> ### 协议细节
>
> #### 使用的命令
>
> 从 hostcontrollercommandids.h 可以看到：
>
> MGMSG_HW_REQ_INFO = 0x0005 *// 请求硬件信息*
>
> MGMSG_HW_GET_INFO = 0x0006 *// 获取硬件信息（响应）*
>
> #### 设备地址
>
> 从 hostcontrolleraddresses.h：
>
> Host       = 0x01 *// 主机地址*
>
> MotherboardController = 0x11 *// 主板控制器*
>
> UsbController   = 0x50 *// USB控制器*
>
> Bay0       = 0x21 *// 机架插槽0*
>
> Bay1       = 0x22 *// 机架插槽1*
>
> ...
>
> Bay9       = 0x2A *// 机架插槽9*
>
> ### 发现过程详解
>
> #### 步骤1：传输层检测
>
> *// 检测USB设备、串口设备等*
>
> iTransportInfo->DetectDataStreamStrings(transportStr, detectedDataStreamStrs);
>
> #### 步骤2：创建临时发现设备
>
> *// 创建一个特殊的"发现设备"，用于查询硬件信息*
>
> iAptDevice = _iAptDeviceFactory->CreateDiscoveryDevice(...);
>
> #### 步骤3：探测控制器地址
>
> *// 先探测 USB Controller (0x50)*
>
> SetRemoteAddress(0x50);
>
> GetHardwareInfo(...); *// 发送 MGMSG_HW_REQ_INFO*
>
> *// 如果失败，探测 Motherboard Controller (0x11)*
>
> SetRemoteAddress(0x11);
>
> GetHardwareInfo(...);
>
> #### 步骤4：解析硬件信息
>
> *// 从响应中获取：*
>
> TLMC_HardwareInfo hardwareInfo {
>
>   serialNumber,   *// 序列号*
>
>   partNumber,    *// 零件号*
>
>   firmwareVersion,  *// 固件版本*
>
>   numChannels    *// 通道数*
>
> };
>
> #### 步骤5：识别设备类型
>
> *// 根据序列号前缀识别设备类型*
>
> uint32_t serialPrefix = serialNumber / 1000000;
>
> *// 查找设备类型映射表*
>
> DeviceTypeDefinitionsBySerialNumberPrefix.find(serialPrefix);
>
> *// 例如：*
>
> *// 27xxxxxx → Kna101*
>
> *// 28xxxxxx → Kdc101*
>
> *// 29xxxxxx → Kpz101*
>
> #### 步骤6：探测机架插槽（如果是主板控制器）
>
> *// 如果是主板控制器，探测所有插槽*
>
> if (address == 0x11) { *// MotherboardController*
>
>   for (byte bay = 0x21; bay <= 0x21 + bayCount; bay++) {
>
> ​    ProbeAptAddress(bay, ...); *// 探测每个插槽*
>
>   }
>
> }
>
> ### 消息格式（APT协议）
>
> 消息头（6字节）：
>
> [消息ID(2字节)] [参数1(1字节)] [参数2(1字节)] 
>
> [源地址(1字节)] [目标地址(1字节)]
>
> 扩展消息：
>
> [消息头(6字节)] + [数据(可变长度)]
>
> 示例：硬件信息查询
>
> 发送：MGMSG_HW_REQ_INFO (0x0005)
>
>    [0x05 0x00] [0x00] [0x00] [0x01] [0x50]
>
>    ↑命令ID  ↑参数 ↑参数 ↑源  ↑目标(USB控制器)
>
> 响应：MGMSG_HW_GET_INFO (0x0006)
>
>    [0x06 0x00] [数据...]
>
>    ↑命令ID  ↑硬件信息数据
>
> ### 总结
>
> 1. 协议：APT (Application Protocol for Thorlabs)
>
> 1. 发现方式：主动探测（发送查询命令）
>
> 1. 关键命令：MGMSG_HW_REQ_INFO (0x0005) / MGMSG_HW_GET_INFO (0x0006)
>
> 1. 探测地址：
>
> - USB Controller: 0x50
>
> - Motherboard Controller: 0x11
>
> - 机架插槽: 0x21-0x2A
>
> 1. 设备识别：通过序列号前缀识别设备类型
>
> 1. 传输层：支持 USB (FTDI) 和串口
>
> ### 核心文件
>
> - discovery/aptdevicediscovery.cpp - APT协议发现实现
>
> - protocol/apt/hostcontrollercommandids.h - 命令ID定义
>
> - protocol/apt/hostcontrolleraddresses.h - 地址定义
>
> - devices/features/hardwareinfo/ - 硬件信息查询功能