# 墨水屏库存便利贴

把清单内容（分栏卡片）渲染成 1bit 位图，通过蓝牙直接推送到墨水屏。

> **第一次看这份，请先打开 `使用指南.md`** —— 那边是按「你想做什么」组织的操作步骤。
> 本文件是技术参考（协议、参数、故障排查）。

- **编辑页面**：`index.html` —— 单文件、零依赖，双击就能开
- **线上地址**：https://lele193.github.io/eink-note/ （HTTPS，手机用 Bluefy 打开可推图）

---

## 一、换新电脑后怎么做

### 1. 拿到这份代码

如果代码在 GitHub 上（推荐，随时能再取）：

```bash
git clone https://github.com/lele193/eink-note.git
cd eink-note
```

只有这份压缩包的话：解压后进入目录即可。

### 2. 直接用（零配置）

双击 `index.html`，用浏览器打开就能编辑、排版、预览。

### 3. 想推到手机 / 推图（需要本地服务器）

iOS 上 Web Bluetooth 只有 Bluefy 提供，且需要 **HTTPS** 或 localhost。
本地 HTTP 只能让手机在同一 Wi-Fi 下访问，**推图功能在纯 http 的局域网地址下不可用**，要用推图请开线上地址。

```bash
python3 serve.py
```

会打印两个地址：

```
本机 http://127.0.0.1:8777/
手机 http://<你的局域网IP>:8777/
```

macOS 自带 python3，无需安装任何东西。

---

## 二、在手机上使用（推荐路径）

1. 从 App Store 安装 **Bluefy**（唯一支持 Web Bluetooth 的 iOS 浏览器）
2. 打开 https://lele193.github.io/eink-note/
3. 点「连接设备」→ 选你的墨水屏

### ⚠️ 缓存问题（重要）

Bluefy 会缓存网页。改动后如果看不到更新，在网址末尾加参数绕过缓存：

```
https://lele193.github.io/eink-note/?b=v18
```

页面右上角会显示当前版本号，对不上就是缓存了。

---

## 三、清单内容保存在哪

存在**浏览器的 localStorage**里，按「浏览器 + 域名」分别独立：

- 手机 Bluefy 的存档 ≠ 电脑浏览器的存档
- 线上地址的存档 ≠ 本地地址的存档

**换电脑时清单不会自动过去**，需要手动搬运：
打开旧电脑的页面 → 展开「文本模式（高级）」→ 全选复制 → 粘到新电脑。

### 多屏支持

每块屏一份清单，按设备名（如 `NRF_EPD_77A3`）自动区分。
连接哪块屏就自动切到那块的档案，推送内容互不干扰。

---

## 四、涉及的固件与硬件

### 设备信息（实测）

- 设备名：`NRF_EPD_77A3`
- 固件：**tsl0922/EPD-nRF5**
  https://github.com/tsl0922/EPD-nRF5
- 屏幕：4.2 寸 400×300，控制器 **SSD1619**，黑白

### 蓝牙协议（已实现在 index.html 里）

| 项 | 值 |
|---|---|
| Service | `62750001-d828-918d-fb46-b6c11c675aec` |
| 写特征 | `62750002-d828-918d-fb46-b6c11c675aec` |
| 版本特征 | `62750003-d828-918d-fb46-b6c11c675aec` |
| 指令 | `SET_PINS 0x00` / `INIT 0x01` / `WRITE_IMG 0x30` / `REFRESH 0x05` |
| 传输流程 | `SET_PINS` → `INIT(驱动 0x04)` → `WRITE_IMG` 分片 → `REFRESH` |

### 两个必须知道的坑

**① 位图极性是反的**

该固件 **`1 = 白，0 = 黑`**，与常见约定相反。写反了会显示反相。

**② 必须先下发驱动**

不先发 `SET_PINS` + `INIT(0x04)`，面板处于未初始化状态，写什么都显示花屏。

---

## 五、目录结构

```
eink-note/
├── index.html      ← 主程序，全部功能都在这一个文件里（1396 行）
├── 使用指南.md      ← 【先看这个】按目的组织的操作步骤
├── README.md       ← 本文件，技术参考
├── check-env.sh    ← 环境自检
├── serve.py        ← 本地服务器（手机同 Wi-Fi 访问用）
├── probe.py        ← BLE 扫描工具：看周围有哪些蓝牙设备
├── probe_deep.py   ← BLE 深度探测：连接设备并 dump 全部服务/特征
└── .gitignore
```

### 蓝牙探测工具（换固件时才需要）

需要额外装 Python 依赖：

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install bleak
```

用法：

```bash
python3 probe.py              # 扫描周围所有蓝牙设备
python3 probe.py NRF_EPD      # 连接该设备并打印全部服务/特征
```

> `bleak` 在 macOS 上通过 CoreBluetooth 工作。极少数情况下 macOS 扫不到某个设备，
> 但手机能连上——这时以手机实测为准，不要根据扫描结果下结论。

---

## 六、部署到 GitHub Pages（可选）

想有自己的线上地址：

```bash
git remote add origin https://github.com/<你的用户名>/<仓库名>.git
git push -u origin main
```

然后到仓库 **Settings → Pages → Source 选 `Deploy from a branch` → 分支 `main`、目录 `/ (root)` → Save**。

等 1–2 分钟，地址就是 `https://<你的用户名>.github.io/<仓库名>/`。

### 注意缓存

Pages 有 CDN 缓存。改动后如果线上没更新，访问 `?b=新版本号` 确认，
同时记得改代码里的 `BUILD` 常量，方便自己核对版本。

---

## 七、代码里能改什么

`index.html` 顶部的 `DEFAULTS` 对象集中了所有固定参数：

```js
const DEFAULTS = {
  pw:400, ph:300,          // 屏幕分辨率，必须与实际一致
  cols:4,                   // 分栏数
  gap:5, pad:6,            // 栏间距 / 整屏边距
  inset:0.05, ipad:0.04,   // 卡片内缩 / 文字左边距
  title:"备货库存",
  tfs:38, tfw:500,         // 标题字号 / 字重
  hdrfull:true,            // 标题底色延伸到屏幕边缘
  hdrstyle:"solid",        // gray=中灰网点底  solid=纯黑底  none=无底
  hdrlinew:15,             // 标题底色高度比例
  cfs:18, ifs:13,          // 分类字号 / 条目字号（会自动缩放）
  lht:145, tgap:3,          // 行距 / 类目与条目额外间距
  autofit:true, nowrap:true, minfs:10,
  numright:true,           // 数量靠右对齐
  mode:"th", th:128        // 二值化：亮度阈值
};
```

推图相关的驱动锁定在：

```js
const EPD = {
  DRV: 0x04,   // 4.2" 黑白 SSD1619 —— 换成你的屏实际的驱动码
  ...
};
```

驱动码对照（来自固件源码）：

| 值 | 屏幕 |
|---|---|
| `0x01` | 4.2寸 黑白 UC8176 |
| `0x04` | 4.2寸 黑白 SSD1619 ← 当前使用 |
| `0x03` | 4.2寸 三色 UC8176 |
| `0x02` | 4.2寸 三色 SSD1619 |
| `0x05` | 4.2寸 四色 JD79668 |

---

## 八、遇到问题

| 现象 | 原因 |
|---|---|
| 页面打开是空白 / 按钮没反应 | 缓存，用 `?b=xxx` 强制刷新 |
| 顶部显示红色「⚠ 脚本出错」 | JS 运行出错，看浏览器控制台 |
| 点连接没反应 | 没用 Bluefy 打开；或页面不是 HTTPS |
| 推上去是花屏 | 驱动码不对（换 `EPD.DRV`）；或漏了 `SET_PINS` |
| 推上去颜色反了 | 位图极性问题（本项目已按 1=白 处理） |
| 清单内容不见了 | 点「恢复上一版」；或从别的设备/地址复制过来 |
| 底部提示有条目换行 | 名称太长，可缩短或减栏数 |

---

## 九、参考链接

- 固件源码：https://github.com/tsl0922/EPD-nRF5
- 官方上位机：https://tsl0922.github.io/EPD-nRF5/
- 排版/推图工具（本项目）：https://github.com/lele193/eink-note
- iOS 蓝牙浏览器：https://apps.apple.com/app/bluefy/id1318626034
