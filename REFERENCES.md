# 参考资料与致谢

本项目的硬件思路、串口协议和 ESPHome 接入方式参考了以下公开资料。`references/` 目录保存的是 2025-07-24 的离线网页快照，仅便于查阅；内容版权归原作者所有。请以原始仓库/文章为准获取最新版本。

## 1. hqc201105 / esphome_tds_sensor

- 原始仓库：https://github.com/hqc201105/esphome_tds_sensor
- 许可证：GPL-2.0（副本见 `references/hqc201105_esphome_tds_sensor_LICENSE.txt`）
- 离线快照：`references/hqc201105_esphome_tds_sensor.mhtml`

感谢 hqc201105 分享 NodeMCU + 双路 BA01 TDS 模块 + TM1637 数码管的 ESPHome 实现，以及 UART 帧结构和显示/开关控制的参考。本项目的 `TDS.yaml` 已针对 ESPHome 2026.11 重写：移除已废弃的 `custom` platform，改为纯 YAML 串口解析，并增加掉电保持、流量换算、TDS 报警和滤芯管理。

## 2. killadm / ESP-WaterMonitor

- 原始仓库：https://github.com/killadm/ESP-WaterMonitor
- 许可证：MPL-2.0（副本见 `references/killadm_ESP-WaterMonitor_LICENSE.txt`）
- 离线快照：`references/killadm_ESP-WaterMonitor.mhtml`

感谢 killadm 分享基于 ESP8266 的双路 TDS/温度与双路流量监测方案，包括 BA01 模块、YF-S402 流量计、接线方式和 Home Assistant 接入范例。本项目沿用“双路 TDS + 双路流量”思路，但按实际使用的 YF-S301 / YF-S402B 型号分别修正了脉冲常数，并把累计流量改为 flash 持久化。

## 3. zonglong / 相館

- 原文：https://zonglong.wordpress.com/2019/09/10/飲用水，水質檢測-（tds，temperature，水流流速/
- 作者：zonglong，发布于 2019-09-10
- 离线快照：`references/zonglong_饮用水水质检测_2019-09-10.mhtml`

感谢 zonglong 记录 BA-01 TDS 模块、YF-S402 霍尔流量计、NodeMCU 与 ESPHome 接入 Home Assistant 的实测过程，并指出低 TDS 区间 BA-01 读数与米家 TDS 笔存在偏差。该偏差提醒：本项目的 TDS 数值适合做趋势和阈值参考，不宜作为计量级水质结论。

## 许可说明

本项目没有逐字复制上述仓库的源代码；`TDS.yaml` 是针对新版 ESPHome 重新编写的配置。参考资料按各自许可证保留归属与许可文本。若后续直接复用上游代码，请遵守对应许可证要求。
