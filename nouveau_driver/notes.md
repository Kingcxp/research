- `nouveau_drm_init()` 初始化 nouveau 驱动
  - 设置默认的 PCI 和平台驱动为空实现
  - 解析和初始化显示相关的选项配置（TODO，里面其实只有一个 `video_nomodeset` 开关）
  - 如果 `nouveau_modeset` 打开(`video_nomodeset` 打开)，则直接返回，表示未启用模式设置
  - 调用 `nouveau_module_debugfs_init()` 初始化 debugfs 用于调试
  - 如果打开了config `CONFIG_NOUVEAU_PLATFORM_DRIVER`，则调用 `platform_driver_register(&nouveau_platform)` 注册平台驱动（用于嵌入式平台等非 PCI 设备）
  - 调用 `nouveau_register_dsm_handler()` 注册 dsm handler
  - 调用 `nouveau_backlight_ctor()` 初始化背光支持
  - 调用 `pci_register_driver(&nouveau_drm_pci_driver)` 注册 PCI 驱动

- `nouveau_drm_exit()` 退出 nouveau 驱动
  - 注销上面注册过的东西

- `nouveau_drm_probe()` 检测并初始化设备
  - 检查是否需要延迟探测
  - 调用 `nvkm_device_pci_new()` 创建 nvkm 设备对象
    - 调用 `pci_enable_device` 启用 PCI 设备
    - 识别设备厂商和型号
    - 分配设备结构体内存
    - 调用 `nvkm_device_ctor()` 初始化设备
      - 映射设备的 MMIO 空间
      - 调用 `nvkm_device_endianness()` 将 mmio 切换到和 CPU 对应的大小端存储方式
        - **读取 0x000004，判断大小端**
        - 如果既不是大端也不是小端，则强制写为大端格式
      - **读取 0x000000 获取设备号，检查设备是否存在**
      - **读取 0x000004 获取 vGPU 是否支持**
      - **读取 0x101000 读取 GPU 芯片类型和架构**
      - 配置时钟频率和初始化子设备
    - 设置 DMA 掩码
  - 调用 `aperture_remove_conflicting_pci_devices()` 移除冲突的帧缓冲驱动
  - 调用 `pci_set_master`() 设置 PCI 主设备
  - 调用 `nouveau_drm_device_new()` 创建 nouveau 设备对象
    - 分配并初始化 nouveau_drm 结构体内存
    - 调用 `drm_dev_alloc()` 分配并初始化 DRM 设备
    - 调用 `dev_set_drvdata()` 和 `nvif_parent_ctor()` 设置设备和驱动之间的关系
    - 调用 `nvif_driver_init()` 初始化 NVIF 接口
    - 调用 `nvif_device_ctor()` 和 `nvif_device_map()` 创建设备对象并映射
    - 调用 `nvif_mclass()` 和 `nvif_mmu_ctor()` 初始化 MMU
  - 调用 `pci_enable_device()` 启用 PCI 设备
  - 调用 `pci_drm_device_init()` 初始化 DRM 设备
  - 根据显存大小，设置显示格式，并调用 `drm_client_setup()` 设置显示缓冲区的颜色格式
  - 调用 `quirk_borken_nv_runpm()` 处理有问题的硬件

**NVKM: Nouveau 内核模式子系统**

Nouveau 驱动的底层硬件抽象层 (HAL)。它负责所有低级（Low-level）的硬件初始化和控制。

- `nvkm_device_init(struct nvkm_device *device)` 初始化 NVIDIA GPU 设备
  - 调用 `nvkm_device_preinit()` 进行设备预初始化
    - 记录开始时间
    - 调用 `nvkm_intr_unarm()` 禁用中断，放置在初始化过程中被中断打扰
    - 遍历所有子设备并执行预初始化 `nvkm_subdev_preinit()`
    - 调用 `nvkm_devinit_post()` 执行设备初始化后处理
    - 调用 `nvkm_top_parse()` 解析设备拓扑结构
    - 调用 `nvkm_fb_mem_unlock()` 解锁显存
    - 计算总耗时并返回
  - 调用 `nvkm_device_fini()` 清理之前的状态
  - 记录初始化开始时间
  - 调用 `nvkm_intr_rearm()` 重新配置中断
    - 检查是否是第一次初始化
    - 如果是第一次，对所有子设备调用 `nvkm_intr_subdev_add()` 添加中断处理
      - 获取设备结构和中断数据
      - 遍历中断数据数组中的每个条目
      - 检查是否为 legacy 模式
      - 根据设备类型调用 `nvkm_intr_subdev_add_dev()` 添加具体的设备
        - 获取子设备对象
        - 设置中断优先级
        - 调用 `nvkm_inth_add()` 添加中断处理
          - 将中断处理程序添加到对应优先级的链表中
        - 调用 `nvkm_inth_allow()` 启用中断处理
          - 使用自旋锁和原子操作确保修改过程安全
          - 检查并更新中断掩码状态
    - 使用自旋锁保护中断配置过程
    - 对每个中断块进行配置：先 block 所有中断，再 allow 需要的中断
    - 调用 `nvkm_intr_rearm_locked()` 完成中断配置
  - 如果设备有特定的初始化函数(`device->func->init`)，则执行
  - 遍历所有子设备并逐个调用 `nvkm_subdev_init` 初始化
  - 调用 `nvkm_acpi_init()` 初始化 ACPI
  - 调用 `nvkm_therm_clkgate_enable()` 启用时钟门控
  - 记录初始化耗时
  - 如果发生任何错误会记录错误信息并清理

- `nvkm_device_del(struct nvkm_device *device)`
  - 安全地释放一个 nvkm_device 对象极其所有相关资源

所有对于 BAR0（MMIO Registers）的 IO（for NV45）：

PMC:

`nv04_mc_init`
- 0x000200: 写入 0xffffffff，完全打开 Engine Master Enable
- 0x001850: 写入 0x00000001，关闭 ROM Access

`nv04_mc_intr`

.pending
- 遍历所有中断叶子节点，获取每个节点的中断状态寄存器
- 0x000100 + (leaf * 4): 读取中断状态寄存器
- 如果任何一个中断状态寄存器不为 0，表示有待处理的中断，设置 pending 为 true。

.unarm
- 遍历所有中断叶子节点，解除中断使能
- 0x000140 + (leaf * 4): 写入 0x00000000，关闭中断使能
- 读取 0x000140，确保之前的写操作完成（强制内存屏障）

.rearm
- 遍历所有中断叶子节点，启用中断
- 0x000140 + (leaf * 4): 写入 0x00000001，启用中断使能

`nv04_mc_device`

.enable
- 0x000200: 将地址改换为 (temp & ~(mask)) | mask

.disable
- 0x000200: 将地址改换为 (temp & ~(mask)) | 0x00000000

.enabled
- 0x000200: 看看是不是和 mask 完全一样

PBUS:

`nv32_bus_init`
- 0x001100: 写入 0xffffffff，中断状态寄存器
- 0x001140: 写入 0x00070008，中断掩码寄存器

`nv32_bus_intr`
- stat: 0x001100 & 0x001140：读取中断状态寄存器和中断掩码寄存器
- 0x001104 & 0x001144：读取 GPIO 状态寄存器和 GPIO 掩码寄存器
- 处理 GPIO 中断
- 处理 MMIO 错误（掩码后的中断状态 & 0x00000008 != 0）
  - 0x009084: 读取错误地址
  - 0x009088: 读取错误错误类型
  - stat &= ~0x00000008
  - 0x001100: 写入 0x00000008，清除中断状态
- 处理温度传感器中断（掩码后的中断状态 & 0x00070000 != 0）
  - stat &= ~0x00070000
  - 0x001100: 写入 0x00070000，清除中断状态
- 如果还有未处理中断，将其报告并清除
  - 0x001100: 将地址改换为 (temp & ~(stat)) | 0x00000000

PCLOCK:

`nv40_clk_read`
- 0x00c040: 读取时钟信息（mast 寄存器）
- 根据时钟源类型返回响应的频率：
  - crystal：返回晶振频率，常量
  - href：返回 100000 （PCIe/AGP 频率）
  - core：从 mast 寄存器获取核心时钟源并计算（read_clk）
  - shader：从 mast 寄存器获取着色器时钟源并计算（read_clk）
  - mem：使用 `read_pll_2` 读取内存时钟

`read_clk`
- 参数：clk - 时钟结构体指针，src - 时钟源选择
- 当 src 为 3 时，使用 `read_pll_2` 读取 0x004000 处的 PLL
- 当 src 为 2 时，使用 `read_pll_1` 读取 0x004008 处的 PLL
- 其他情况返回0

`read_pll_1` - 读取并计算第一类锁相环的输出频率
- 读取目标寄存器的控制值
- 分别从其中提取分频系数 P、倍频系数 N 和 参考分频系数 M

`read_pll_2` - 读取并计算第二类锁相环的输出频率
- 读取目标寄存器的32位控制值
- 读取目标寄存器的后32位系数值
- 分离系数并计算

`nv40_clk_calc` - 计算并设置显卡的核心时钟和着色器时钟的 PLL 配置参数
- 计算核心时钟的 PLL 参数（调用 `nv40_clk_calc_pll`）
- 根据计算结果配置 `npll_ctrl` 和 `npll_coef` 寄存器
- 如果着色器时钟与核心时钟不同，同样计算

`nv40_clk_calc_pll`
- 从 BIOS 中解析 PLL 信息
- 如果目标频率低于 VCO1 的最大频率，则禁用 VCO2
- 调用 `nv04_pll_calc` 计算具体的 PLL 参数
- 返回结果

`nv40_clk_prog` - 设置显卡的时钟频率
- 通过一系列寄存器操作来配置时钟参数：
  - 0x00c040：时钟控制寄存器
  - 0x004004：NPLL系数寄存器
  - 0x004000：NPLL控制寄存器
  - 0x004008：SPLL寄存器
- 使用nvkm_mask和nvkm_wr32函数进行寄存器操作
- mdelay(5)提供5毫秒的延时，确保时钟稳定
- 重新启用时钟控制

`nv40_clk_tidy` - 清理时钟配置
- 什么也没做（？）

devinit:

`nv04_devinit_preinit`
- 0x000200: 通过 nvkm_mask 函数启用 I2C 总线访问
- 检查并保存当前 VGA 所有权状态
  - 对于 NV45，直接调用 `nvkm_rdvgac` 读取 0x44 寄存器的值
    - 调用 `nvkm_wrport`，之前传入的 0x44 作为 data，head 是 0，port 是 0x03d4
      - 根据端口类型分为 3 类：
        - AR：属性寄存器 (port = 0x03c0 | 0x03c1)
        - INP0：输入状态 0 (port = 0x03c2 | 0x03da)
        - CR：CRT 控制器 (port = 0x03d4 | 0x03d5)
      - 如果是上面的三种，向 0x601000 + (head * 0x2000) + port 写入 data
      - 否则应是另外 3 类：
        - MISC：杂项寄存器 (port = 0x03c2 | 0x03cc)
        - SR：序列寄存器 (port = 0x03c4 | 0x03c5)
        - GR：图形寄存器 (port = 0x03ce | 0x03cf)
      - 如果是上面的三种，向 0x0c0000 + (head * 0x2000) + port 写入 data
    - 调用 `nvkm_rdport`，head 是 0，port 是 0x03d5
      - 和 `nvkm_wrport` 类似，只不过只需要读取
  - 对于 NV45，直接调用 `nvkm_wrvgac` 写入VGA控制器的所有权寄存器（0x44）
- 将VGA所有权设置为0，解除CRT控制器的从属状态
  - 调用 `nvkm_wrport`，之前传入的 0x44 作为 data，head 是 0，port 是 0x03d4
  - 调用 `nvkm_wrport`，data 为 0，head 是 0，port 是 0x03d5
- 读取多个VGA寄存器来获取水平总像素数(htotal)
- 如果htotal为0，说明显示器未初始化，需要执行POST(Power-On Self-Test)

`nv04_devinit_pll_set` - 设置 NVIDIA 显卡的 PLL 频率
- 从 BIOS 中解析配置信息
- 计算达到目标频率的 PLL 参数
- 根据芯片版本选择合适的 PLL 设置方法
  - 读取主 PLL 寄存器当前值
    - 传入参数 reg1，读取 32 位寄存器值
  - 读取次 PLL 寄存器当前值
    - reg2 = reg1 + ((reg1 == 0x680520) ? 0x5c : 0x70)
    - 读取位于 reg2 的 32 位寄存器值
- 0x680580: 读取 RAMDAC 控制寄存器
- 0x001584: 读取电源控制寄存器 1，存为 saved_powerctrl_1
- 0x001584: 修改电源控制寄存器 1，设置电源控制位为 (saved_powerctrl_1 & ~(0xf << shift_powerctrl_1)) | 1 << shift_powerctrl_1)
- 0xc040: 读取控制寄存器，存为 savedc040
- 0xc040: 修改控制寄存器，设置控制位为 savedc040 & ~(3 << shift_c040)
- 0x680580: 更新 RAMDAC 控制寄存器
- 更新次 PLL 寄存器
- 更新主 PLL 寄存器
- 恢复电源控制寄存器 1 的原始值
- 恢复控制寄存器 c040 的原始值

PFB:

`nv20_fb_tags` - 获取帧缓冲标签
- 0x100320: 读取帧缓冲标签

`nv40_fb_init` - 初始化帧缓冲
- 0x10033c: 清除特定位(0x00008000)

`nv20_fb_tile_prog` - 配置显卡帧缓冲的分块模式，参数 i 是块区域索引
- 0x100244 + (i * 0x10): 设置限制
- 0x100248 + (i * 0x10): 设置步长
- 0x100240 + (i * 0x10): 设置地址
- 0x100240 + (i * 0x10): 读取地址寄存器
- 0x100300 + (i * 0x04): 设置压缩参数

`nv40_ram_new` - 初始化显存
- 根据显卡寄存器确定显存类型和大小
  - 读取 0x001218，判断显存类型
  - 读取 0x10020c，判断显存大小
- 根据读取的值创建响应的显存对象
- 设置显存的分区数量
  - 读取 0x100200，设置显存分区数

GPIO:

`nv10_gpio_intr_stat` - GPIO 中断状态处理
- 读取并处理 GPIO 中断
  - 0x001104: 读取 intr 寄存器
  - 0x001144: 读取 intr_mask 寄存器
  - stat = intr & intr_mask
- 将中断状态分为高位和低位两部分
- 0x001104: 重新写入 intr（怎么没修改还写？）

`nv10_gpio_intr_mask` - GPIO 中断掩码处理
- 0x001144: 读取中断屏蔽寄存器到 inte
- 根据 type 参数，分别处理高电平和低电平的中断屏蔽
```c
if (type & NVKM_GPIO_LO)
  inte = (inte & ~(mask << 16)) | (data << 16);
if (type & NVKM_GPIO_HI)
  inte = (inte & ~mask) | data;
```
- 将修改后的 inte 值写回中断屏蔽寄存器

`nv10_gpio_drive` - 控制 GPIO 引脚
- 根据不同的 GPIO 引脚号范围，使用不同的寄存器和掩码
- 将方向和输出值编码到 data 变量中
- 使用nvkm_mask函数修改相应的寄存器
```c
static int
nv10_gpio_drive(struct nvkm_gpio *gpio, int line, int dir, int out)
{
	struct nvkm_device *device = gpio->subdev.device;
	u32 reg, mask, data;

	if (line < 2) {
		line = line * 16;
		reg  = 0x600818;
		mask = 0x00000011;
		data = (dir << 4) | out;
	} else
	if (line < 10) {
		line = (line - 2) * 4;
		reg  = 0x60081c;
		mask = 0x00000003;
		data = (dir << 1) | out;
	} else
	if (line < 14) {
		line = (line - 10) * 4;
		reg  = 0x600850;
		mask = 0x00000003;
		data = (dir << 1) | out;
	} else {
		return -EINVAL;
	}

	nvkm_mask(device, reg, mask << line, data << line);
	return 0;
}
```

`nv10_gpio_sense` - 检测 GPIO 引脚的输入状态
- 根据不同的 GPIO 引脚号范围，使用不同的寄存器和掩码
  - line 0-1：从0x600818寄存器读取，每个引脚占用16位
  - line 2-9：从0x60081c寄存器读取，每个引脚占用4位
  - line 10-13：从0x600850寄存器读取，每个引脚占用4位

instmem:

`nv40_instmem_oneinit` - 初始化内存空间
- 0x001540: 获取特定字段（& 0x0000ff00），获得顶点着色器数量
- 根据顶点着色器和 GPU 版本计算内存空间大小
- 添加额外的空间用于 PCI(e)GART 表和对象存储
- 对齐到 4KB 边界
- 创建内存堆，分配内存区域

PPCI:

`nv40_pci_msi_rearm`
- 调用 `nvkm_pci_wr08(pci, 0x0068, 0xff)`
  - 调用 `nvkm_wr08(pci->subdev.device, pci->func->cfg.addr + 0x0068, 0xff)`

PTIMER:

`nv41_timer_init` - 配置 GPU 定时器
- 获取设备晶振频率
- 计算分频器参数 (m, n, d)
- 将计算好的参数写入硬件寄存器
  - 0x009220: 写入 m - 1
  - 0x009200: 写入 n
  - 0x009210: 写入 d

`nv41_timer_intr` - 处理 GPU 定时器中断
- 检查是否报警
  - 0x009100: 读取中断状态寄存器，记作 stat
- 如果 stat & 0x00000001
  - 0x009100: 写入 0x00000001
  - 触发报警
  - stat &= ~0x00000001
- 如果 stat 仍不为 0
  - 记录错误信息
  - 0x009100: 写入 stat

`nv40_timer_read` - 读取 GPU 定时器值
- 0x009410: 读取高32位
- 0x009400: 读取低32位

`nv04_timer_time` - 设置 GPU 定时器时间
- 0x009410: 写入高32位
- 0x009400: 写入低32位

`nv04_timer_alarm_init` - 初始化定时报警器
- 0x009420: 写入报警时间值
- 0x009140: 写入 0x00000001 启用定时器中断

`nv04_timer_alarm_fini` - 关闭定时报警器
- 0x009140: 写入 0x00000000 禁用定时器中断

disp:

`nv04_disp_intr` - 显示中断处理函数
- 0x600100: 读取 crtc0，阴极射线管控制器 0 的中断状态
- 0x602100: 读取 crtc1，阴极射线管控制器 1 的中断状态
- 分别检查是否有垂直中断发生
  - 如果发生，对对应地址写入 0x00000001

fifo:

`nv40_fifo_init` - 初始化 FIFO 队列
- 配置基本参数
  - 0x002040: 写入 0x000000ff，设置 FIFO 的延迟/超时值
  - 0x002044: 写入 0x2101ffff，设置 DMA 获取的超时和时间片参数
  - 0x002058: 写入 0x00000001，Unknown
- 配置 FIFO 内存哈希表
  - 0x002210: 配置 RAMHT，在 VRAM 中的地址和大小
  - 0x002218: 配置 RAMRO，
- 0x002220: 配置 RAMFC 的基地址
- 0x003204: 设置最大通道 ID 或通道掩码
- 0x002100: 清除所有未决的中断状态
- 0x002140: 启用所有中断
- 0x003200: 启用 PFIFO 推送机制
- 0x003250: 启用 PFIFO 拉取机制
- 0x002500: 启用 FIFO 缓存重平衡，表示初始化完成
