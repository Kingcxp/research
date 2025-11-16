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
    - 设置 DMA 掩码
  - 调用 `aperture_remove_conflicting_pci_devices()` 移除冲突的帧缓冲驱动
  - 调用 `pci_set_master` 设置 PCI 主设备
  - 调用 `nouveau_drm_device_new` 创建 nouveau 设备对象
  - 调用 `pci_enable_device` 启用 PCI 设备
  - 调用 `pci_drm_device_init` 初始化 DRM 设备
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
