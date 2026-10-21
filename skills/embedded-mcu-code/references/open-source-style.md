# 开源风格基准

本基准用于校准固定产品的实现取舍，不是第三方编码规范，也不要求复制第三方源码。当前项目、工具链和用户约定优先。评审固定以下来源版本，避免随主分支变化重新定义目标。

## 1. 主参考：Grbl

仓库：gnea/grbl；发布标签：v1.1h.20190825；提交：40eb439bf2afc4c6f3003836c396e183ba72c45b。定位是 ATmega328P 的专用 CNC 控制固件，作为直接实现的风格参考，不作为当前 MCU 驱动模板或维护活跃度推荐。

| 来源 | 观察与参考边界 |
| --- | --- |
| [protocol.c：89–104、141–147](https://github.com/gnea/grbl/blob/40eb439bf2afc4c6f3003836c396e183ba72c45b/grbl/protocol.c#L89-L104) | 定长行缓冲、入口容量处理、具名命令调用。职责边界由实际工作决定，不要求回调注册；该文件还承担实时协调，不照搬其全部职责。 |
| [gcode.c：61–75、340起、839起](https://github.com/gnea/grbl/blob/40eb439bf2afc4c6f3003836c396e183ba72c45b/grbl/gcode.c#L61-L75) | 内部入口遵循已规范化输入约定，命令语义先校验再执行。减少重复防御不等于删除命令合法性判断；不照搬千行函数和共享解析状态。 |
| [motion_control.c：28–40](https://github.com/gnea/grbl/blob/40eb439bf2afc4c6f3003836c396e183ba72c45b/grbl/motion_control.c#L28-L40) | 数据对象按真实需要用指针，不机械判空；统一运动入口承担软限位和运行状态检查。动作允许条件不同于参数合法性。 |
| [coolant_control.c：81–126](https://github.com/gnea/grbl/blob/40eb439bf2afc4c6f3003836c396e183ba72c45b/grbl/coolant_control.c#L81-L126) | 模式按值传入，具名控制接口，sync有真实时序作用；简单接口并不排斥有意义的同步函数。 |

不采用其两空格/混合制表符、总入口 grbl.h、大量跨模块全局状态，以及 AVR 寄存器和原子性假设。串口 ISR 中有特殊实时命令路径，不能把它推广为普通业务放进中断。等待规划器空间的协作循环也不能当作本产品总线等待模板。

## 2. 补充参考：Betaflight

仓库：betaflight/betaflight；发布标签：2026.6.2；提交：e0b7bb01b17b21351057e9ead2d1ab39dd44fa16。重点参考测量、业务状态和有效协议交付的职责组织，不采用其整套多硬件兼容架构。

| 来源 | 观察与参考边界 |
| --- | --- |
| [voltage.c：161–213](https://github.com/betaflight/betaflight/blob/e0b7bb01b17b21351057e9ead2d1ab39dd44fa16/src/main/sensors/voltage.c#L161-L213) | 换算、采样刷新、结果读取各有作用；持续值和滤波历史缓存有用途。配置/结果指针直接使用，未对这些内部参数逐项判空。 |
| [battery.c：282–365](https://github.com/betaflight/betaflight/blob/e0b7bb01b17b21351057e9ead2d1ab39dd44fa16/src/main/sensors/battery.c#L282-L365) | 滞回、持续时间、告警和低压策略在电池业务模块，主流程调用有意义的私有函数；枚举状态与 switch 可以直接表达工况。 |
| [msp_serial.c：178–218、413–442、528–533](https://github.com/betaflight/betaflight/blob/e0b7bb01b17b21351057e9ead2d1ab39dd44fa16/src/main/msp/msp_serial.c#L178-L218) | 长度、校验和、完整包交付分工；有多个实际串口和协议版本，具名端口上下文与处理函数有真实用途。我们只有固定调用关系时采用具名直接调用，不复制这些兼容机制。 |
| [voltage.c：271–278](https://github.com/betaflight/betaflight/blob/e0b7bb01b17b21351057e9ead2d1ab39dd44fa16/src/main/sensors/voltage.c#L271-L278) | 可能没有 ESC 数据时检查结果指针，过期数据也有处理；与已知有效内部参数的机械判空不同。 |

不采用其 camelCase、_t 类型命名、多传感器来源、PG 配置注册框架、大量条件编译和完整串口回调架构。测量模块也依赖电池配置等产品信息，并非严格单向分层；参考局部职责，不把它描述成零耦合范例。

## 3. 后续比较的固定问题

- 能否从主流程看出输入、处理、状态更新和动作？
- 改变换算、业务门限或命令动作时，修改是否落在负责该规则的位置？
- 指针、缓存、上下文、回调和同步函数是否对应实际数据量、生命周期、实例或时序？
- 是否按边界处理输入与故障，同时避免内部相同条件层层重复检查？

功能和时序正确是前提；风格是上述取舍的表现，不按文件数、代码行数、判空或函数指针数量排名。后续用于调整的任务与用于验收的任务分开，比较前固定这些问题；出现新风险时说明新增依据，不为了支持某个输出临时改标准。
