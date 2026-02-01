本文完整记录本次 `Failed to initialize NVML: Driver/library version mismatch` 问题的**现象、环境、逐步骤排查逻辑、命令拆解、根因、修复方案**，可作为同类问题的标准化排查手册，下次遇到直接按流程复现解决。

## 1. 问题基础信息
### 1.1 运行环境
- 系统：Ubuntu 24.04
- 显卡：NVIDIA GA100 [A800 80GB PCIe]（共 8 块）
- 用户权限：普通用户`zhihao`，**无 `sudo/root`  管理员权限**
- 内核态 NVIDIA 驱动版本：`580.95.05`
- 报错提示 NVML 库版本：`580.126`

### 1.2 问题现象
执行`nvidia-smi`、`nvidia-smi --version`均抛出相同错误，无法查看显卡状态、算力、显存等任何信息：
```bash
➜ zhihao nvidia-smi 
Failed to initialize NVML: Driver/library version mismatch NVML library version: 580.126

➜ zhihao nvidia-smi --version 
Failed to initialize NVML: Driver/library version mismatch NVML library version: 580.126
```

## 2. 前置核心原理（必须理解）

排查前先明确两个核心组件，这是所有操作的理论基础：
- **NVRM（内核态驱动模块）**

    - 运行在内核空间，负责和显卡硬件直接交互，加载后版本固定，路径信息：`/proc/driver/nvidia/version`
    - 本次内核版本：`580.95.05`
    
- **NVML（用户态库）**
    
    - 全称为 NVIDIA Management Library，是用户态的调用接口，`nvidia-smi`工具**完全依赖该库**实现功能
    - 报错中`580.126`是系统记录的用户态 NVML 库版本
    
- **铁律**：内核态 NVRM 版本 + 用户态 NVML 库版本**必须严格匹配**，且 NVML 库文件必须存在且可被工具找到，否则直接触发版本不匹配报错。