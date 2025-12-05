---
marp: true
theme: gaia
size: 16:9
paginate: true
math: katex
backgroundColor: #FFFFFF
color: #303133
footer: NV45 GPU Simulation & Driver Interaction Report
style: |
  p {
    margin: 16px;
    font-size: 28px;
  }

  section {
    position: relative;
    font-family: Bahnschrift;
    padding-bottom: 64px !important;
    padding-top: 64px !important;
  }

  section.title-page {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }

  section::before {
    content: "";
    position: absolute;
    background-color: #76b900;
    height: 64px;
    width: 100%;
    top: 0;
    left: 0;
  }

  section::after {
    content: "";
    background-color: #F2F3F5;
    margin-top: auto;
    height: 64px;
    width: 100%;
    display: flex;
    flex-direction: row-reverse;
    z-index: 0;
  }

  header {
    z-index: 1;
    color: #FFFFFF;
    font-weight: bold;
  }

  footer {
    z-index: 1;
    color: #303133;
  }

  img {
    max-height: 65vh;
    max-width: 100%;
  }

  strong {
    color: #76b900 !important;
  }

  code {
    background-color: #f0f0f0;
    color: #d63384;
    padding: 2px 5px;
    border-radius: 4px;
  }

  pre {
    background-color: #303133;
    color: #ffffff;
    padding: 10px;
    border-radius: 8px;
  }

  td {
    font-size: 32px;
  }
---

<!-- _class: title-page -->

## NV45 GPU Simulation

Analysis of Nouveau Driver & Hardware Emulation Strategy

---

<!-- 
_class: title-page 
_header: Hardware Architecture
-->

## Part 1:
## **NV45 Hardware Architecture**

###### Understanding the Target Device

---

<!-- _header: NV45 (Curie) Internal Structure -->

The NV45 GPU consists of multiple sub-devices ("Engines") coordinated by the Master Control.

![](assets/architecture.png)

---

<!-- _header: Memory Interface (BARs) -->

The CPU communicates with the GPU via three **Base Address Registers (BARs)**.

| BAR | Type | Size | Description | Method |
| :--- | :--- | :--- | :--- | :--- |
| **BAR 0** | **MMIO** | 16 MB |  Registers for all engines. | `readl` / `writel` (Intercepted by QEMU) |
| **BAR 1** | **VRAM** | 512 MB | Textures, Framebuffer, PushBuffers. | Direct Memory Access / TTM Mapping |
| **BAR 2** | **PRAMIN** | 1 MB | Channel status, Hash Tables. | Direct Access (Stored as Structs) |

---

<!-- _header: Sub-Devices -->

1.  **bios** - **BIOS** (Basic Input/Output System)
    * Responsible for handling the graphics card's BIOS information, including initialization parameters and hardware configuration.
2.  **bus** - **Bus Interface**
    * Responsible for managing the communication interface between the **GPU** and the system bus.
3.  **clk** - **Clock Controller**
    * Responsible for clock generation, distribution, and frequency adjustment for various components within the **GPU**.

---

4.  **devinit** - **Device Initialization**
    * Responsible for the initial setup and configuration of the **GPU** hardware.
5.  **fb** - **Framebuffer**
    * Responsible for managing video memory (VRAM) and the framebuffer, handling the storage of graphics data.
6.  **gpio** - **General Purpose Input/Output**
    * Responsible for controlling the general-purpose pins on the **GPU**, used for various hardware control functions.

---

7.  **i2c** - **I2C Bus Controller**
    * Responsible for the **I2C** communication protocol, used to communicate with displays and other peripherals.
8.  **imem** - **Instruction Memory**
    * Responsible for managing the **GPU**'s instruction storage area, storing the microcode to be executed.
9.  **mc** - **Memory Controller**
    * Responsible for managing **GPU** memory access and memory mapping.

---

10. **mmu** - **Memory Management Unit**
    * Responsible for virtual-to-physical address translation, managing the **GPU**'s address space.
11. **pci** - **PCI Interface Controller**
    * Responsible for managing the interface and communication between the **GPU** and the **PCI** bus.
12. **therm** - **Thermal Management**
    * Responsible for monitoring **GPU** temperature and managing the cooling system.

---

13. **timer** - **Timer**
    * Responsible for providing timing functions and a time base.
14. **volt** - **Voltage Regulator**
    * Responsible for the regulation and management of the **GPU** core voltage.
15. **disp** - **Display Controller**
    * Responsible for managing display output, including signal generation and display mode control.

---

16. **dma** - **Direct Memory Access**
    * Responsible for managing direct memory transfers between various **GPU** components.
17. **fifo** - **Command FIFO** (First In, First Out)
    * Responsible for managing and scheduling the **GPU** command queue, ensuring commands are executed in order.
18. **gr** - **Graphics Rendering Engine**
    * Responsible for processing **3D** graphics rendering and **2D** graphics acceleration.

---

19. **mpeg** - **MPEG Decoder**
    * Responsible for hardware **MPEG** video decoding acceleration.
20. **sw** - **Software Objects**
    * Responsible for managing objects and operations at the software level.

---

<!-- _header: The Fuzzing Loop (IOCTL Flow) -->

How a fuzzing payload reaches the simulated GPU.

![](assets/payload-sequence.png)

---

<!--
header: The Whole Process: Analysis
_class: title-page
-->

## The Whole Process: Analysis
**The overall process of simulating a GPU driver**
