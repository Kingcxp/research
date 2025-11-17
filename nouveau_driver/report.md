## Linux Nouveau 驱动代码阅读笔记

### nouveau_drm.c - 驱动主入口文件

##### 模块初始化

- `nouveau_drm_init()` - 注册 PCI 驱动
- `nouveau_drm_exit()` - 注销 PCI 驱动

##### PCI 设备初始化

- `nouveau_drm_probe()` - 探测到支持的 GPU 后调用
- `nouveau_drm_remove()` - 当设备被移除时调用

##### 设备生命周期管理

- `nouveau_drm_device_init()` - 初始化 DRM 设备和各个子系统
- `nouveau_drm_device_fini()` - 清理和关闭设备

##### 电源管理

- `nouveau_pmops_suspend()` - 挂起设备
- `nouveau_pmops_resume()` - 恢复设备
- `nouveau_pmops_runtime_suspend/resume()` - 运行时电源管理

##### 文件操作接口

- `nouveau_drm_open()` - 应用程序打开设备文件时调用
- `nouveau_drm_postclose()` - 应用程序关闭设备文件时调用
- `nouveau_drm_ioctl()` - 处理来自用户空间的 IOCTL 请求

### nvkm/engine/device/base.c - 设备核心管理

- `nvkm_device_ctor()` - 构造设备对象
- `nvkm_device_init()` - 初始化所有子设备和引擎
- `nvkm_device_find()` - 查找设备
- `nvkm_device_fini()` - 关闭和清理设备

### nvkm/engine/device/pci.c - 设备识别和生命周期

### nouveau_drv.h - 驱动核心头文件

### nvif/device.c - 设备接口层

- `nvif_device_ctor()`
