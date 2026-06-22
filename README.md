# ADBSweep 工具箱

一款基于 WPF 的安卓系统应用管理工具箱，内置丰富的包名数据库，帮助用户安全地识别和管理安卓预装应用。

## 🧰 项目组成

| 项目 | 类型 | 说明 |
|------|------|------|
| **AdbSweep** | WPF 桌面应用 | 主工具，通过 ADB 连接安卓设备，查看/停用/启用/卸载系统应用 |
| **AdbSweepEditor** | WPF 桌面应用 | 包名数据库可视化编辑器，用于维护 `packages.json` 数据 |

---

## 🔧 AdbSweep — 主工具

### 功能概览

- **ADB 设备连接**：智能检测 ADB 路径（内置 platform-tools → 本地 → PATH → Android SDK），自动识别设备品牌/型号/序列号
- **系统应用列表**：通过 `adb shell pm list packages -s` 获取全部系统应用，按安全等级着色
- **应用操作**：停用（推荐）/ 启用 / 卸载 / 打开应用 / 备份 APK
- **右键菜单**：快速操作 + 搜索（Bing / 秘塔 AI / 纳米 AI）+ 复制包名
- **操作历史**：记录所有操作，支持一键恢复全部已停用应用
- **热更新**：启动时自动从 GitHub 拉取最新包名数据库，无需更新软件
- **数据导出**：按安全等级筛选导出 JSON
- **命令行参数**：支持 `--console` 交互模式、`--auto-disable` 批量禁用

### 安全等级分类

| 等级 | 颜色 | 含义 | 停用 | 启用 | 卸载 |
|------|------|------|------|------|------|
| **Forbidden** | 🔴 红色 | 核心系统进程 | ❌ 硬拦截 | ❌ 按钮禁用 | ❌ 硬拦截 |
| **Caution** | 🟡 黄色 | 系统服务/框架/引擎 | ⚠️ 1次确认 | ✅ 直接执行 | 🚫 2次确认 |
| **Basic** | 🔵 蓝色 | 基础功能应用 | ⚠️ 1次确认 | ✅ 直接执行 | 🚫 2次确认 |
| **Safe** | 🟢 绿色 | 独立应用 | ✅ 直接执行 | ✅ 直接执行 | ⚠️ 2次确认 |
| **Unknown** | 🟠 橙色 | 未收录包名 | ⚠️ 1次确认 | ✅ 直接执行 | 🚫 2次确认 |

### 热更新机制

```
启动 → 读取云端 version → 比本地新？→ 下载覆盖缓存 → 完成
                               ↓
                             版本相同 → 跳过
                               ↓
                             网络失败 → 静默使用本地缓存 + 内置兜底
```

**代理加速**：设置中可填入任意第三方代理前缀（如 `https://ghfast.top/`），自动拼接默认 GitHub Raw 源：

```
{代理前缀} + raw.githubusercontent.com/.../packages.json
```

留空则直连 GitHub Raw。用户可根据网络环境自由配置。

### 数据加载策略

```
优先级 1 → 云端缓存 JSON（packages_cache.json），含完整 5 级分类
优先级 2 → 内置硬编码兜底
                ├── Forbidden 级别的核心包名（约30条，确保离线时也阻止破坏性操作）
                └── 关键词映射（约120+，从未收录包名智能推测中文名）
                └── 其余包名 → 标记为 Unknown（橙色），同步数据后恢复正常分类
```

> **内置仅保留最核心的防护数据**，完整数据库（Caution/Basic/Safe 全部分类）通过热更新从云端获取。

---

## ✏️ AdbSweepEditor — 包名数据库编辑器

### 功能概览

- **可视化编辑**：直观编辑包名中文名称、安全等级、功能简述、停用后果
- **实时自动保存**：修改等级、编辑文本失焦时自动写回 `packages.json`
- **搜索与筛选**：按包名/中文名搜索，按安全等级/核验状态筛选
- **核验标记**：每个条目可标记为"已核验"，跟踪数据审核进度
- **重复包名处理**：导入时自动检测并让用户选择保留哪个版本
- **快捷预设**：预设用途描述文本，一键追加到原因字段
- **右键操作**：快速设置安全等级、复制包名、在线搜索、删除条目
- **导航与快捷键**：上下导航、`Ctrl+S` 导出、`Ctrl+Z` 还原

### 使用场景

1. **ADBSweep** 连接设备 → 同步云端数据 → 导出设备包名 JSON
2. **AdbSweepEditor** 导入上述 JSON → 逐条核验数据准确性 → 标记"已核验"
3. 未收录包名（Unknown）→ 设置等级、填写中文名称和停用后果 → 导出
4. 将编辑后的 `packages.json` 提交 PR 至本仓库

---

## 📦 数据格式

```json
{
  "version": 1,
  "updatedAt": "2026-06-20",
  "packages": {
    "包名": {
      "name": "中文名称",
      "level": "安全等级",
      "reason": "停用/卸载后的影响说明"
    }
  },
  "keywords": {
    "英文关键词": "对应的中文名称"
  }
}
```

### 字段说明

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `version` | int | ✅ | 版本号，每次更新 +1（触发客户端热更新） |
| `updatedAt` | string | ✅ | 更新日期，`YYYY-MM-DD` 格式 |
| `packages` | object | ✅ | 包名字典，key 为安卓完整包名 |
| `packages.<key>.name` | string | ✅ | 包的中文名称 |
| `packages.<key>.level` | string | ✅ | `Forbidden` / `Caution` / `Basic` / `Safe` / `Unknown` |
| `packages.<key>.reason` | string | ✅ | 停用或卸载后的影响说明 |
| `keywords` | object | ✅ | 关键词→中文名映射，用于智能识别未知包名 |

### 示例

```json
{
  "version": 1,
  "updatedAt": "2026-06-20",
  "packages": {
    "com.android.systemui": {
      "name": "系统界面",
      "level": "Forbidden",
      "reason": "核心系统界面进程，停用后状态栏/导航栏/锁屏全部失效"
    },
    "com.miui.cloudservice": {
      "name": "小米云服务",
      "level": "Safe",
      "reason": "小米云同步服务，不需要云同步可停用"
    },
    "com.example.unknown": {
      "name": "未知应用",
      "level": "Unknown",
      "reason": "未收录的包名，需人工审核"
    }
  },
  "keywords": {
    "launcher": "桌面启动器",
    "cloud": "云服务"
  }
}
```

---

## 🚀 构建

### 环境要求

- [.NET 10.0 SDK](https://dotnet.microsoft.com/download) 或更高版本
- Windows 10+ (x64)

```bash
# 主工具
cd AdbSweep
dotnet publish -c Release -r win-x64 --self-contained false

# 编辑器
cd AdbSweepEditor
dotnet publish -c Release -r win-x64 --self-contained false
```

发布输出到 `bin/Release/net10.0-windows/win-x64/publish/` 目录。

---

## 🤝 如何贡献

### 方式一：使用编辑器（推荐）

1. 下载 [AdbSweepEditor](https://github.com/cayuensui/ADBSweep/releases) 编辑器
2. 打开云端最新的 `packages.json`
3. 新增或修正包名数据，自动保存到 JSON
4. 提交 PR 至本仓库

### 方式二：直接编辑 JSON

1. **Fork** 本仓库
2. 编辑 `packages.json`：
   - 在 `packages` 中添加新条目（按字母顺序排列）
   - 如有需要，在 `keywords` 中添加新关键词
   - **将 `version` 字段 +1**
   - 更新 `updatedAt` 为当前日期
3. 提交 PR，标题格式：`[新增] 品牌名 - 简述` 或 `[修正] 包名 - 修正内容`

### PR 示例

```
[新增] 三星 - Galaxy S24 系列预装应用
[新增] vivo - OriginOS 4 新增系统应用
[修正] com.xiaomi.market - 修正分类从 Safe 改为 Basic
[新增] 新增 15 个 OPPO ColorOS 14 系统包名
```

### 格式检查清单

提交前请确认：

- [ ] JSON 格式有效（可用 [JSONLint](https://jsonlint.com/) 验证）
- [ ] 包名使用完整的安卓包名格式（如 `com.xiaomi.xxx`）
- [ ] `level` 为有效值：`Forbidden` / `Caution` / `Basic` / `Safe` / `Unknown`
- [ ] `name` 简洁明了，不包含品牌名（如用「云服务」而非「小米云服务」）
- [ ] `reason` 清楚说明停用/卸载的实际影响
- [ ] `version` 已 +1，`updatedAt` 已更新
- [ ] 新条目按字母顺序排列在正确位置

---

## 📂 项目结构

```
adb_tool/
├── README.md
├── packages.json               # 安卓系统包名数据库（云端数据源）
├── AdbSweep/                   # 主工具 — WPF 桌面应用
│   ├── MainWindow.xaml/.cs     # 主界面
│   ├── Models/                 # 数据模型
│   ├── Services/               # ADB/历史/更新/设置服务
│   └── platform-tools/         # 内置 ADB 工具
├── AdbSweepEditor/             # 数据库编辑器 — WPF 桌面应用
│   ├── MainWindow.xaml/.cs     # 编辑器界面
│   ├── Models/                 # 数据模型
│   └── Converters/             # 值转换器
```

## 📄 许可

本工具箱仅供学习和研究用途。包名数据来源于公开信息收集和社区贡献。
