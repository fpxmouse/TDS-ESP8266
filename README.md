# TDS / 流量监测 ESP8266 配置手册

本目录的 `TDS.yaml` 是给 Home Assistant ESPHome Device Builder 使用的最终配置，适用于 NodeMCU ESP8266 + 双路 TDS 模块 + 双流量计 + TM1637 数码管。

> 安全提示：本项目可公开发布。OTA 密码和 Fallback AP 密码均通过 `!secret` 读取，请把真实凭据只放在本地 `secrets.yaml`，不要提交该文件。

## 1. 功能概览

- 双路 TDS 与温度：`Raw Water TDS` / `Purified Water TDS`，对应原水和纯水。
- 数码管：只显示 TDS，每 3 秒在进水/纯水之间切换；`TDS Switch` 关闭时熄灭。
- 出水 TDS 超标报警：`Purified Water TDS Alarm` 与 `Purified Water TDS Excess`。
- 双路流量：`Raw Water Flow Rate` / `Purified Water Flow Rate`。
- 累计流量：`Total Raw Water Usage` / `Total Purified Water Usage`，断电保持。
- 今日流量：`Today's Raw Water Usage` / `Today's Purified Water Usage`，跨天自动归零。
- 五级滤芯寿命：周期可设、上次更换日期可设，掉电保持，并有到期提醒。
- 维护按钮：`Reset Water Totals`、`Restart ESP`、五个滤芯的 `Reset (Replaced)`。
- 汇总实体：`Purifier Status Summary`，方便在 HA 中做通知。

## 2. 硬件接线

| 功能 | 引脚 | 说明 |
|---|---|---|
| TM1637 CLK | D1 | 数码管时钟 |
| TM1637 DIO | D2 | 数码管数据 |
| TDS 模块 UART | TX / RX | 9600 baud |
| 进水流量计 | D6 | YF-S301，1260 脉冲/升 |
| 纯水流量计 | D5 | YF-S402B，1920 脉冲/升 |

两个流量计型号不同，换算常数不能混用。若更换型号，需要同步修改 `TDS.yaml` 中 `in_rate` / `in_session_total` 和 `out_rate` / `out_session_total` 的 `filters` 常数。

## 3. 安装到 Home Assistant

1. 打开 HA → 设置 → 加载项 → **ESPHome Device Builder**。
2. 进入 ESPHome Web UI，新建或打开名为 `tds-esp8266` 的设备配置。
3. 用本目录 `TDS.yaml` 的内容替换配置；HA 版本中的 API 加密部分若使用 `!secret tds_esp8266__encryption_key`，保留 HA 原来那一行即可。
4. 在同一目录准备 `secrets.yaml`，至少包含：

```yaml
wifi_ssid: "你的WiFi名称"
wifi_password: "你的WiFi密码"
ota_password: "请替换为足够长的随机密码"
fallback_ap_password: "请替换为备用热点密码"
```

可直接复制 `secrets.example.yaml` 作为起点。

5. 点击 **Validate** 验证配置。
6. 编译通过后，用 USB 首次烧录；之后可在 HA 里直接 OTA 升级。

## 4. 首次烧录与 OTA

- 首次烧录：用 ESPHome Web UI 的 **Install** → 选择串口。
- OTA 升级：设备在线后，用 **Install** → **Over the air**；也可命令行：

```bash
esphome upload tds-esp8266.yaml --device 10.10.50.173
```

- OTA 后若 HA 连不上，通常是本机还有 `esphome logs` 占用了 ESP8266 的单个原生 API 连接；关闭日志窗口再刷新设备即可。
- 若设备当前固件仍使用旧 OTA 密码（例如 `1`），第一次 OTA 前先把 `ota_password` 设为旧值；升级成功后再改成强密码并再次 OTA，否则会认证失败。
- `compile_process_limit: 1` 是为了避免 HA 主机内存不足时多线程编译被 OOM 杀掉。

## 5. 掉电保持机制

ESP8266 默认把 `restore_value` 存在 RTC memory，完全断电会丢。本配置做了两层处理：

- `esp8266.restore_from_flash: true`
- 关键 globals 显式 `storage: flash`

断电保持的数据包括：累计总流量、今日基准、TDS 报警阈值、滤芯寿命与更换日期。

`g_last_session_*` 不持久化，重启后从 0 开始，与 `pulse_counter` 的 session total 对齐；30 秒周期只把新增量并入总量，避免重复计算。

## 6. 常用操作

### 修改 TDS 报警阈值

在 HA 中找到 `Purified Water TDS Alarm Threshold`，默认 50 ppm。修改后掉电仍保留。

### 设置滤芯

- `Filter N Life Days`：设置寿命天数。默认：PP棉 90 天、GAC 180 天、CTO 180 天、RO 膜 730 天、后置碳 365 天。
- `Filter N Last Replace Date`：直接填写上次更换日期。
- `Filter N Reset (Replaced)`：按下后把日期记为今天。

`Filter N Used Days` 自动计算已用天数，`Filter N Due` 在达到寿命时变为 on。

### 清零累计流量

按下 `Reset Water Totals`。它会把总量、今日基准和 session 并入基准一起复位，避免清完又被 30 秒周期加回去。

### 数码管显示开关

`TDS Switch` 打开时显示 TDS，关闭时熄灭。显示逻辑在 `display.lambda` 中，按 `millis() / 3000` 在两路 TDS 间切换。

## 7. 已知注意事项

- 配置使用 `!secret wifi_ssid` / `!secret wifi_password`，没有本地 WiFi 凭据时无法编译。
- 不要把本地 `secrets.yaml` 提交到仓库；OTA 和 Fallback AP 凭据应使用强密码。
- TDS 串口帧解析假设帧头 `55 0A 85`、11 字节帧；若换 TDS 模块固件，需重新抓 UART 验证。
- SNTP 未成功同步前，滤芯天数会显示 0，这是正常保护行为。
- 本配置面向 ESPHome 2026.11+；旧版 ESPHome 可能不支持 `compile_process_limit` 或 `storage: flash` 写法。

## 8. 参考资料与致谢

硬件思路、UART 协议和 ESPHome 接入方式参考了多个公开项目与文章。作者、原始链接、许可证和离线快照说明见 [REFERENCES.md](REFERENCES.md)。

## Support / 请作者喝杯咖啡

如果这份配置帮你省了时间，欢迎扫码请作者喝杯咖啡。二维码已隐去姓名和头像，可直接扫码。

<table>
  <tr>
    <td align="center"><b>支付宝</b><br><img src="support/alipay-qr.svg" width="260" alt="Alipay QR Code"></td>
    <td align="center"><b>微信支付</b><br><img src="support/wechat-pay-qr.svg" width="260" alt="WeChat Pay QR Code"></td>
  </tr>
</table>

