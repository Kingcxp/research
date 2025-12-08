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
    max-height: 55vh;
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
    font-size: 28px;
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

---

### Lifecycle: From Boot to Shutdown

The interaction between **Nouveau** and **NV45** follows a strict state machine.

![](assets/state-machine.png)

1. **Probe**: Identification & PCI Enable.
2. **Init**: Hardware Reset & Engine Wake-up.
3. **Runtime**: IOCTLs, Command Submission, Interrupts.
4. **Exit**: Resource Cleanup & Shutdown.

---

### Detecting the Hardware

The driver validates the device identity via **BAR0** reads.

![](assets/detection.png)

---

### Initializing Engines

`nvkm_device_init` wakes up sub-devices sequentially.

| Step | Engine | Action | Key Register |
| :--- | :--- | :--- | :--- |
| **1** | **MC** | Disable Interrupts & Enable Engines | `0x000140` (Intr), `0x000200` (Master) |
| **2** | **TIMER** | Calibrate Timing & Divider | `0x009220`, `0x009400` |
| **3** | **FB** | Detect VRAM Size & Partitions | `0x10020c` (Size), `0x100200` (Count) |
| **4** | **FIFO** | Setup RAMFC/RAMHT & Enable | `0x003200` (Push), `0x003250` (Pull) |
| **5** | **GR** | Clear Context & Enable Pipe | `0x40032c`, `0x40013c` |

---

### The Timer Handshake

QEMU **must** respond to timer reads, or the driver will hang (deadlock).

![](assets/timer-handshake.png)

* **Logic**: The driver writes frequency dividers ($m, n, d$) and expects the timer counter at `0x009400` to increment immediately.

---

### Handling User Commands

When `Mesa` or user-space sends a draw command:

![](assets/handling-commands.png)

1.  **Push**: Commands written to shared memory.
2.  **Kick**: Driver updates the `PUT` pointer (Doorbell).
3.  **Pull**: GPU fetches commands between `GET` and `PUT`.

---

### The Interrupt Cycle

How the GPU signals completion or errors.

![](assets/interrupt.png)

---

### Safe Shutdown

When `nouveau_drm_exit` is called, the hardware must be quieted safely.

1.  **Block Interrupts**: Write `0` to `0x000140` (MC).
2.  **Stop FIFO**: Disable Push/Pull caches at `0x003250`.
3.  **Unmap**: Release BAR mappings.

> **Note**: Failure to stop the FIFO before unloading can cause the GPU to write to freed memory.

---

<!-- _class: title-page -->

## **Sub-Device Specifics**

###### Detailed Register Interactions & Responsibilities

---

### MC (`nv17_mc`): The Central Hub

**Role**: Manages global interrupt routing and engine states.

* **Initialization**:
    * **Enable Engines**: Writes `0xffffffff` to **`0x000200`** to wake up all units.
    * **ROM Access**: Writes `0x00000001` to **`0x001850`** to disable ROM visibility.
* **Interrupt Handling (`nv04_mc_intr`)**:
    * **Status**: Reads **`0x000100`** (+ leaf offsets) to identify which engine (FIFO, PGRAPH, etc.) triggered an IRQ.
    * **Rearm**: Writes `1` to **`0x000140`** to re-enable interrupts after handling.

---

### TIMER (`nv41_timer`): Synchronization

**Role**: Provides a stable nanosecond timebase for driver delays and timeouts.

* **Calibration (`nv41_timer_init`)**:
    * Driver calculates divider values ($m, n, d$) based on crystal frequency.
    * Writes to **`0x009220`** (m-1), **`0x009200`** (n), **`0x009210`** (d).
* **Runtime Operation**:
    * **Read Time**: Returns 64-bit timestamp via **`0x009400`** (Low) and **`0x009410`** (High).
    * **Alarms**: Writes alarm target to **`0x009420`**. When reached, triggers IRQ at **`0x009100`**.

---

### FB (`nv40_fb`): VRAM Management

**Role**: Manages VRAM aperture, tags, and memory compression settings.

* **Probe & Setup (`nv40_ram_new`)**:
    * **Detect Type**: Reads **`0x001218`** to identify memory generation.
    * **Detect Size**: Reads **`0x10020c`** to determine total VRAM size.
    * **Partitions**: Reads **`0x100200`** to configure memory controllers.
* **Configuration**:
    * **Tags**: Accesses **`0x100320`** for Framebuffer Tag memory.
    * **Tile Programming**: Configures memory tiling regions (Limit/Pitch/Address) at **`0x100240`** series.

---

### DEVINIT: The Bootstrap

**Role**: Handles legacy VGA compatibility and Clock (PLL) setup.

* **VGA Ownership (`nv04_devinit_preinit`)**:
    * Arbitrates ownership via register **`0x44`**.
    * Uses `nvkm_wrport` to unlock CRTC registers (Port **`0x03d4`**).
* **Clock Setup (`nv40_clk_prog`)**:
    * Configures **PLLs** (Phase Locked Loops) to set Core/Memory frequencies.
    * Manipulates **`0x00c040`** (Control) and **`0x004000`** (NPLL).
    * *Crucial for Simulation:* QEMU must emulate the PLL locking behavior or the driver will timeout waiting for clocks to stabilize.

---

### FIFO (`nv40_fifo`): The Workhorse

**Role**: Fetches commands from DMA/VRAM and pushes them to PGRAPH.

![](assets/fifo.png)

* **Initialization (`nv40_fifo_init`)**:
    * **Timeouts**: Configures retry limits at **`0x002040`**.
    * **Context Structures**: Sets RAMHT location (**`0x002210`**) and RAMFC base (**`0x002220`**).
* **Activation**:
    * Enables **Push** (**`0x003200`**) and **Pull** (**`0x003250`**) engines.

---

### FIFO: Runtime Management

The FIFO engine is heavily interrupt-driven for context switching.

* **Interrupt Handling (`nv04_fifo_intr`)**:
    * **Reassign**: Reads **`0x002500`** to handle cache reassignment requests.
    * **Semaphores**: Checks **`0x00326c`** for synchronization primitives.
    * **Channel ID**: Reads **`0x003204`** to verify which channel is currently active.
* **Pausing**:
    * To safely switch contexts, the driver writes **`0`** to **`0x002500`** (Disable Cache) and **`0x003250`** (Stop Puller).

---

### GR (`nv40_gr`): The Rendering Core

**Role**: Executes the actual 3D/2D drawing commands.

* **Initialization (`nv40_gr_init`)**:
    * **Sanitize**: Clears Context Register **`0x40032c`**.
    * **Debug Config**: Sets specific pipeline behavior via **`0x400084`** and **`0x40008c`**.
    * **Pipe Control**: Configures internal pipeline states at **`0x400820`**.
* **Exception Handling**:
    * If an invalid command is sent, PGRAPH raises an interrupt at **`0x400100`**.
    * The driver reads this to kill the offending channel without crashing the kernel.

---

### Register Map Summary (QEMU Focus)

| Block | Offset Range | Critical Function |
| :--- | :--- | :--- |
| **MC** | `0x000000 - 0x001FFF` | Interrupts, Device ID, Endianness |
| **FIFO** | `0x002000 - 0x003FFF` | Push/Pull Control, RAMHT, RAMFC |
| **TIMER**| `0x009000 - 0x009FFF` | Timebase, Alarms |
| **FB** | `0x100000 - 0x100FFF` | VRAM Config, Tile Regions |
| **GR** | `0x400000 - 0x40FFFF` | Graphics Context, Pipeline Config |
| **CRTC** | `0x600000 - 0x602FFF` | Display Output (Head 0/1) |

*Simulation Strategy*: Focus on implementing **MC**, **TIMER**, and **FIFO** logic first. GR can initially be a "black box" that consumes data without rendering.
