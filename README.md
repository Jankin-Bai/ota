# XiaoT Card OTA Firmware Repository

OTA 固件仓库，用于 XiaoT Card（GR5526 + Cat1 Air780EGH 双 MCU 架构）的远程升级。

## 目录结构

```
ota/
├── cat1/                    # Cat1 (Air780EGH) 固件
│   ├── cat1_ota_latest.bin # Cat1 OTA 升级包（内含 GR5526 固件）
│   ├── version.json         # 版本信息
│   └── checksums.txt        # 校验和
└── gr5526/                  # GR5526 固件
    └── gr5526_fw_latest.bin # GR5526 OTA 固件（单独升级用）
```

## 固件说明

### Cat1 OTA 包 (cat1/cat1_ota_latest.bin)

- **格式**: LuatOS LuaDB 脚本文件系统镜像
- **大小**: ~297 KB
- **内容**: Cat1 应用脚本 + `/luadb/gr5526_fw.bin`（GR5526 固件资源）
- **升级方式**: 通过 `CAT1_OTA_DL=<url>` AT 命令触发下载，`CAT1_OTA_START` 执行升级

### GR5526 固件 (gr5526/gr5526_fw_latest.bin)

- **格式**: GR5526 OTA 固件（含 Image Info 头）
- **大小**: ~223 KB
- **升级方式**: Cat1 通过 UART DFU 传输给 GR5526，或单独通过 J-Link 烧录

## OTA 升级流程

### Cat1 自升级

1. 将 `cat1_ota_latest.bin` 放到 HTTP 服务器
2. 通过 GR5526 CLI 或 AT 口发送：
   ```
   CAT1_OTA_DL=http://your-server/cat1_ota_latest.bin
   CAT1_OTA_START
   ```
3. Cat1 下载固件并写入 Flash，自动重启

### GR5526 自动升级（通过 Cat1）

1. Cat1 OTA 包内含 `/luadb/gr5526_fw.bin`
2. Cat1 开机后自动检测该文件，通过 CRC32 版本标记防重复
3. 自动通知 GR5526 进入 DFU 模式，通过 UART 传输固件
4. 升级成功后写入标记，下次开机不再重复升级

## 版本信息

当前版本：`001.000.000`
构建时间：2026-09-07

## 硬件

- **GR5526**: BLE 5.3 SoC，双 Bank Flash（Bank A @0x240000, Bank B @0x2C0000）
- **Cat1**: Air780EGH，LuatOS-SoC V2034，脚本区 384KB
- **通信**: UART4 @ 460800 bps
