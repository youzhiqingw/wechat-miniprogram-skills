# 微信小程序 UI 组件库选型指南

## 官方推荐组件库对比

| 组件库 | 维护方 | 适用场景 | 包体积 | 基础库要求 | 更新频率 |
|--------|--------|---------|--------|-----------|---------|
| **WeUI** | 微信官方 | 追求微信原生体验、快速开发 | 扩展库引入（0体积） | 2.2.3+ | 随框架更新 |
| **TDesign** | 腾讯多 BG 联合 | 企业级应用、多端统一 | ~100KB（按需引入） | 2.6.0+ | 持续迭代 |
| **CloudBase Agent UI** | 腾讯云开发 | AI 对话场景、智能体应用 | ~50KB | 2.13.0+ | 高频更新 |

---

## WeUI 组件库

### 简介
WeUI 是微信官方设计团队为微信小程序量身设计的组件库，与微信原生视觉体验完全一致。

### 特点
- ✅ 扩展库引入，**不占用小程序包体积**
- ✅ 视觉与微信原生完全一致
- ✅ 官方长期维护，稳定性高
- ✅ 适合表单、列表、弹窗等基础页面

### 安装方式

**方式一：扩展库引入（推荐）**
```json
// app.json
{
  "useExtendedLib": {
    "weui": true
  }
}
```

```json
// page.json
{
  "usingComponents": {
    "mp-dialog": "weui-miniprogram/dialog/dialog",
    "mp-button": "weui-miniprogram/button/button",
    "mp-cells": "weui-miniprogram/cells/cells",
    "mp-cell": "weui-miniprogram/cell/cell"
  }
}
```

**方式二：npm 引入**
```bash
npm install weui-miniprogram
```

### 官方资源
- [微信官方文档](https://developers.weixin.qq.com/miniprogram/dev/platform-capabilities/extended/weui/)
- [GitHub 源码](https://github.com/wechat-miniprogram/weui-miniprogram)
- [快速开始文档](https://wechat-miniprogram.github.io/weui/docs/quickstart.html)

---

## TDesign 组件库

### 简介
TDesign 是腾讯官方推出的企业级设计体系，覆盖 Vue/React/小程序/Flutter 等多端，提供统一的设计语言和组件实现。

### 特点
- ✅ 企业级设计规范，视觉统一
- ✅ 支持多端（小程序 + H5 + 桌面端）
- ✅ 组件丰富，可定制性强
- ✅ TypeScript 支持完善

### 安装方式

```bash
npm install tdesign-miniprogram
```

```json
// app.json
{
  "usingComponents": {
    "t-button": "tdesign-miniprogram/button/button",
    "t-cell": "tdesign-miniprogram/cell/cell",
    "t-input": "tdesign-miniprogram/input/input"
  }
}
```

### 官方资源
- [TDesign 官网](https://tdesign.tencent.com/)
- [小程序版 GitHub](https://github.com/Tencent/tdesign-miniprogram)
- [开发者工具集成指引](https://developers.weixin.qq.com/miniprogram/dev/devtools/other-resource.html#tdesign)

---

## CloudBase Agent UI

### 简介
CloudBase Agent UI 是腾讯云开发团队提供的 AI 对话场景专用组件库，可快速搭建微信小程序 AI 聊天界面。

### 特点
- ✅ 快速集成 AI 对话能力
- ✅ 支持混元/DeepSeek 等大模型
- ✅ 内置文件上传、语音输入、联网搜索
- ✅ 支持 Agent 智能体工具调用

### 安装方式

```bash
npm install @cloudbase/ai-agent-ui
```

```json
// app.json
{
  "usingComponents": {
    "agent-ui": "@cloudbase/ai-agent-ui/miniprogram/agent-ui"
  }
}
```

### 配置要求

1. **配置 request 合法域名**
   ```
   https://{envId}.api.tcloudbasegateway.com
   ```

2. **云开发环境初始化**
   ```javascript
   // app.js
   App({
     onLaunch() {
       wx.cloud.init({
         env: 'your-env-id',
         traceUser: true
       })
     }
   })
   ```

### 官方资源
- [官方文档](https://docs.cloudbase.net/ai/agent-ui/agent-ui-mp)
- [GitHub 源码](https://github.com/TencentCloudBase/cloudbase-agent-ui)

---

## 选型建议

### 按场景选择

| 场景 | 推荐组件库 | 理由 |
|------|-----------|------|
| 快速原型开发 | WeUI | 零配置、原生体验、不占用包体积 |
| 企业级应用 | TDesign | 设计规范统一、组件丰富、可定制 |
| AI 对话应用 | CloudBase Agent UI | 专为 AI 场景设计、开箱即用 |
| 多端统一项目 | TDesign | 一套设计体系覆盖多端 |

### 按包体积选择

```
WeUI（扩展库）: 0 KB（不计入包体积）
CloudBase Agent UI: ~50 KB
TDesign（按需引入）: ~100 KB
```

### 注意事项

1. **基础库版本要求**
   - WeUI: 2.2.3+
   - TDesign: 2.6.0+
   - CloudBase Agent UI: 2.13.0+

2. **包体积控制**
   - 主包大小限制：2MB
   - 总包大小限制：20MB（含分包）
   - 建议使用扩展库或按需引入

3. **兼容性检查**
   ```javascript
   // 检查基础库版本
   const { SDKVersion } = wx.getSystemInfoSync()
   console.log('当前基础库版本:', SDKVersion)
   ```

---

## 链接验证状态（截至 2024-2025）

| 链接 | 状态 | 验证时间 |
|------|------|---------|
| `https://tdesign.tencent.com/` | ✅ 有效 | 2025-10 |
| `https://developers.weixin.qq.com/miniprogram/dev/platform-capabilities/extended/weui/` | ✅ 有效 | 2024-2025 |
| `https://weui.io/` | ✅ 有效 | 持续维护 |
| `https://docs.cloudbase.net/ai/agent-ui/agent-ui-mp` | ✅ 有效 | 2025-2026 |
| `https://wechat.design/tool/weui-mobile` | ⚠️ 已归档 | 建议使用替代入口 |

---

## 相关文档

- [版本兼容性指南](./version-compatibility.md) - 检查基础库版本要求
- [部署指南](./deployment-guide.md) - 域名配置、备案、隐私协议
