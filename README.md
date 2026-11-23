# SanaSta · Galgame 游玩数据统计

把本地 Excel 里记录的 galgame 评分表，变成一个能看、能改、能出图的桌面工具。

- 读你已有的表格（三个工作表：评分表 / 最喜爱女主 / 女主统计），**双向同步**：在程序里改评分会写回原表
- 自动从 [VNDB](https://vndb.org) 抓封面、评分、厂商、主线女角色与声优（日文原名）
- 六张统计图（厂商分布、两组声优分布、评分区间、发色分布、厂商均分），一键导出 PNG
- 作品卡片可导出为分享图（含本作最喜爱女主、主观调整与备注），整库可导出成一张总表大图
- 备注、删除作品（带二级确认与连带清理）、批量导入、伪登录（自定义头像与用户名）
- 写回直接改原表，但每次先自动备份；只对目标单元格做 XML 补丁，**保留 WPS 的单元格图片与共享公式**

Tauri 2 + Rust 重写，单文件 **8.4 MB**，空闲内存约 36 MB（原 Electron 版 316 MB / 472 MB）。

## 下载安装

到 [Releases](https://github.com/aLKaphRah/SanaSta/releases) 下载最新 `SanaSta-x.y.z-Setup.exe`，双击安装即可。

- 安装到当前用户目录，**不需要管理员权限**
- 自动检测并安装 **WebView2 运行时**（Win10 1903+ / Win11 通常已自带）
- 已安装旧版时直接运行新安装包即完成升级，配置与数据保留
- 卸载从「设置 → 应用」或开始菜单里的卸载项；**不会删除你的数据**（数据在 `%APPDATA%\SanaSta`）

⚠️ **不要把程序放在 `C:\dsh` 目录下运行**（实测该目录会让程序无法写任何外部文件，包括表格所在目录），安装器默认位置已避开。

## 界面

| 页面 | 作用 |
|---|---|
| 作品库 | 封面、VNDB 评分、厂商、八项评分条与总评；总评下方灰色显示**全库排名** |
| 最喜爱女主 | 表 2 卡片视图：立绘、声优日文原名、角色 ID |
| 统计总览 | 六张图表，支持单张/全部导出（导出图带标题） |
| 立绘图墙 | 按发色浏览立绘，最爱女主带浅绿描边；点图可改判发色 |
| 匹配管理 | 表格名字 ↔ VNDB 条目绑定，支持候选选择与手动填 VNDB ID |
| 设置 | 表格路径、备份与恢复、自定义权重与分制（同步改写表头与公式） |

作品详情抽屉里可以拖滑块改分、查看雷达图、直接选择/更换本作最喜爱女主，并把该作品**导出成一张分享卡片**：

![作品卡片](docs/card.png)

## 数据放哪

| 位置 | 内容 |
|---|---|
| 你的 xlsx | 表格本体（可放在任意可写位置） |
| `%APPDATA%\SanaSta\config.json` | 表格路径、权重/分制、统计口径 |
| `%APPDATA%\SanaSta\cache.json` | VNDB 数据缓存（离线可用） |
| `%APPDATA%\SanaSta\mapping.json` | 表格名字 ↔ VNDB ID 绑定 |
| `%APPDATA%\SanaSta\backups\` | 每次写回前的自动备份（保留 30 份） |
| `%APPDATA%\SanaSta\cache\images\` | 封面与立绘缓存（便携模式：`SANASTA_DATA_DIR` 可改） |

首次运行若找不到表格，会在数据目录生成一份空的标准表（三张工作表 + 表头 + 总评公式）。

## 开发

```
galstats/
  renderer/            界面唯一来源（HTML/CSS/JS + ECharts）
  tauri/
    sync-ui.js         把 renderer 同步到 tauri/ui 并注入 tauri-shim.js
    src-tauri/
      src/zipxlsx.rs   xlsx 解压 / XML 手术式补丁 / 重打包（含 WPS DISPIMG）
      src/model.rs     维度权重与公式、三张表解析
      src/patcher.rs   改分、新建作品、女主卡片、图墙增删移
      src/stats.rs     厂商/声优/评分分布/发色聚合
      src/store.rs     配置、缓存、映射、备份、图片
      src/vndb.rs      VNDB Kana API 客户端（限速 + 429 退避 + 发色归类）
      src/template.rs  首次运行生成空表
      src/commands.rs  全部 IPC 命令
```

构建：

```bash
cd tauri/src-tauri
cargo build --release            # 产物 target/release/sanasta.exe
```

打包安装器（需要 Inno Setup 6）：

```bash
cd galstats
ISCC.exe build/installer.iss     # 产物 output/SanaSta-<version>-Setup.exe
```

## 许可

MIT
