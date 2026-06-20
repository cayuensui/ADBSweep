# ADBSweep - 安卓系统包名数据库

为 [ADBSweep 工具箱](https://github.com/cayuensui/ADBSweep) 提供**热更新**的系统包名数据，帮助用户安全地识别和停用安卓系统预装应用。

## 📊 安全等级分类

| 等级 | 标识 | 含义 | 说明 |
|------|------|------|------|
| **Forbidden** | 🔴 禁止 | 核心系统进程 | 停用/卸载将导致手机故障，**严禁操作** |
| **Caution** | 🟡 谨慎 | 系统服务/框架/引擎 | 停用可能导致部分功能异常，需二次确认 |
| **Basic** | 🟠 基础 | 基础功能应用 | 电话/短信/时钟/相机/联系人等，可停用但强烈不建议卸载 |
| **Safe** | 🟢 安全 | 独立应用 | 可自由停用，不影响系统正常运行 |

### 分类原则

- **Forbidden**：`android`（系统框架）、`systemui`（系统界面）、`settings`（设置）等硬件抽象层和核心进程
- **Caution**：输入法引擎、蓝牙/ NFC 中间件、运营商配置、打印服务等非必要但影响功能的组件
- **Basic**：拨号、通讯录、相机、日历、闹钟等用户可自行安装替代品的基础应用
- **Safe**：厂商预装的第三方应用、云服务客户端、游戏、演示程序等

## 📦 数据格式

```json
{
  "version": 1,
  "updatedAt": "2026-06-20",
  "packages": {
    "包名": {
      "name": "中文名称",
      "level": "安全等级",
      "reason": "停用/卸载后果说明"
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
| `packages` | object | ✅ | 包名字典，key 为安卓包名 |
| `packages.<key>.name` | string | ✅ | 包的中文名称 |
| `packages.<key>.level` | string | ✅ | 安全等级：`Forbidden` / `Caution` / `Basic` / `Safe` |
| `packages.<key>.reason` | string | ✅ | 停用或卸载后的影响说明 |
| `keywords` | object | ✅ | 关键词映射，帮助工具智能识别未知包名 |

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
    }
  },
  "keywords": {
    "launcher": "桌面启动器",
    "cloud": "云服务"
  }
}
```

## 🤝 如何贡献

### 提交新包名 / 修正数据

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
- [ ] `level` 为四个有效值之一：`Forbidden` / `Caution` / `Basic` / `Safe`
- [ ] `name` 简洁明了，不包含品牌名（如用「云服务」而非「小米云服务」）
- [ ] `reason` 清楚说明停用/卸载的实际影响
- [ ] `version` 已 +1，`updatedAt` 已更新
- [ ] 新条目按字母顺序排列在正确位置

## 🔄 热更新机制

ADBSweep 工具箱启动时会自动检查本仓库的 `version` 字段：

```
启动 → 读取云端 version → 比本地新？→ 下载覆盖缓存 → 完成
                                ↓
                              版本相同 → 跳过
                                ↓
                              网络失败 → 静默使用本地数据
```

用户也可在设置中配置 CDN 加速源（jsDelivr / Staticaly / Fastly）以改善国内网络环境。

### CDN 加速直链

如果 GitHub Raw 访问不稳定，可以使用以下 CDN 替换默认源：

| CDN | URL 格式 |
|-----|----------|
| GitHub Raw（默认） | `https://raw.githubusercontent.com/cayuensui/ADBSweep/main/packages.json` |
| jsDelivr | `https://cdn.jsdelivr.net/gh/cayuensui/ADBSweep@main/packages.json` |
| Staticaly | `https://cdn.staticaly.com/gh/cayuensui/ADBSweep/main/packages.json` |

## 📄 许可

本数据库仅供学习和研究用途。包名数据来源于公开信息收集和社区贡献。
