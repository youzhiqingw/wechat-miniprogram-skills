# 微信小程序开发 Skills 项目

## 项目概述

这是一个专为 AI 编程助手设计的**微信小程序开发技能包**（Skill），提供全面的开发指导、性能优化方案和最佳实践。该项目不是传统意义上的软件项目，而是一个**知识库和文档集合**，用于指导 AI 助手在微信小程序开发场景中提供专业、高质量的技术支持。

### 核心目标

- 让 AI 助手成为微信小程序开发专家
- 提供高质量的代码和架构建议
- 识别并解决性能问题和兼容性问题
- 遵循微信小程序的最佳实践和开发规范

### 技术栈覆盖

- ✅ 原生开发（JavaScript/TypeScript）
- ✅ 跨平台框架（Taro/Uni-app）
- ✅ 云开发（微信云开发/CloudBase）
- ✅ 构建工具（微信开发者工具/miniprogram-ci）

---

## 项目结构

```
wechat-miniprogram-skills/
├── SKILL.md                    # 核心技能文件（AI 助手的主要指导文档）
├── README.md                   # 项目说明和使用指南
├── QWEN.md                     # 本项目上下文说明（本文件）
├── assets/
│   └── images/                 # 图片资源
├── examples/                   # 示例代码和模板
│   ├── basic-template.md       # 基础模板示例
│   ├── cloud-development.md    # 云开发完整示例
│   ├── typescript-template.md  # TypeScript 项目示例
│   └── basic-project/          # 基础项目结构示例
└── references/                 # 参考文档
    ├── api-reference.md        # API 速查手册
    ├── deployment-guide.md     # 发布与部署指南
    ├── error-handling.md       # 错误处理与错误码对照表
    ├── performance-optimization.md  # 性能优化详解
    ├── testing-guide.md        # 测试与调试指南
    ├── ui-components-guide.md  # UI 组件库选型指南
    └── version-compatibility.md     # 基础库版本兼容性指南
```

---

## 关键文件说明

### 1. `SKILL.md`（核心文件）

这是整个项目的**核心技能文件**，包含：

- **5 大核心原则**：性能优先、原生兼容、代码质量、用户体验、安全规范
- **场景定义**：明确何时使用此 Skill（小程序开发）和不适用场景（Web/H5/原生应用）
- **项目结构规范**：标准目录结构和关键配置文件说明
- **开发规范**：命名规范、JavaScript/TypeScript 规范、WXML/WXSS 规范、组件化规范
- **性能优化**：setData 优化、渲染优化、分包加载、代码体积优化
- **常见问题防坑指南**：iOS 日期格式、键盘遮挡、页面栈管理、原生组件层级、异步竞态条件、本地存储限制
- **API 使用规范**：网络请求封装、用户授权处理、登录流程
- **云开发集成**：云函数调用、云数据库操作、云存储
- **基础库版本兼容性**：API 版本检查和降级方案
- **测试与调试**：单元测试、自动化测试、真机调试
- **错误处理**：统一错误处理方案

### 2. `examples/` 目录

提供可直接参考的示例代码：

- **basic-template.md**：最小化小程序模板，包含 app.js、app.json、app.wxss、页面文件
- **cloud-development.md**：云开发完整示例，包括云函数、云数据库、云存储
- **typescript-template.md**：TypeScript 项目配置和使用示例

### 3. `references/` 目录

详细的参考文档集合：

- **api-reference.md**：微信小程序 API 速查手册，涵盖基础 API、用户相关、网络请求、数据存储、位置、媒体、交互反馈、导航、界面、设备、转发分享、性能监控、更新管理等
- **performance-optimization.md**：性能优化详解
- **version-compatibility.md**：基础库版本兼容性指南
- **testing-guide.md**：测试与调试指南
- **error-handling.md**：错误处理与错误码对照表
- **deployment-guide.md**：发布与部署指南（含备案、隐私协议配置）
- **ui-components-guide.md**：UI 组件库选型指南（WeUI/TDesign/CloudBase）

---

## 核心知识点

### 性能优化重点

#### setData 优化
```javascript
// ✅ 推荐：使用数据路径局部更新
this.setData({
  'userInfo.nickName': 'New Name',
  'list[0].status': 'completed'
})

// ❌ 避免：全量更新
this.setData({
  userInfo: { ...this.data.userInfo, nickName: 'New Name' }
})
```

#### 性能指标
- 首屏渲染时间：< 2s
- setData 单次数据：< 1024KB
- setData 调用频率：< 10 次/秒
- 代码包总大小：< 20MB
- 主包大小：< 2MB
- 单个分包大小：< 2MB
- 页面栈深度：≤ 10 层

### 常见问题解决方案

#### iOS 日期格式兼容性
```javascript
// ❌ iOS 不支持
new Date('2024-03-31')

// ✅ 兼容写法
new Date('2024/03/31')
```

#### 页面栈管理
```javascript
const pages = getCurrentPages()
if (pages.length >= 10) {
  wx.redirectTo({ url: '/pages/target/target' })
} else {
  wx.navigateTo({ url: '/pages/target/target' })
}
```

---

## 使用方式

此项目主要用于：

1. **AI 助手技能配置**：将 `SKILL.md` 内容复制到 AI 编程助手的技能配置中
2. **开发参考**：在微信小程序开发过程中查阅 `references/` 目录的文档
3. **学习示例**：参考 `examples/` 目录中的模板和示例代码
4. **最佳实践查询**：当遇到小程序开发问题时，根据 `SKILL.md` 中的指导获取解决方案

### 典型使用场景

- 创建、修改或优化微信小程序项目
- 实现小程序页面、组件或功能模块
- 集成微信 API（支付、登录、分享、授权等）
- 配置小程序项目（`app.json`, `project.config.json`）
- 调试、测试、预览或发布小程序
- 性能优化和问题排查
- 云开发集成（云函数、云数据库、云存储）
- 处理平台兼容性问题

---

## 开发规范摘要

### 命名规范
- **文件命名**：小写 kebab-case（如 `user-info.js`）
- **变量/函数**：驼峰 camelCase（如 `getUserInfo`）
- **组件名**：短横线连接（如 `custom-button`）
- **常量**：大写下划线（如 `API_BASE_URL`）
- **样式类名**：BEM 规范（如 `.user-info__avatar--large`）

### 代码规范
- 使用箭头函数处理 this 绑定
- 使用 async/await 处理异步操作
- 使用解构赋值
- 使用 const/let，避免使用 var
- 避免过长的函数（不超过 50 行）

### WXML 规范
- `wx:for` 必须包含 `wx:key`
- 频繁切换使用 `hidden`，不频繁切换使用 `wx:if`
- 合理使用事件冒泡（`bindtap` vs `catchtap`）

### WXSS 规范
- 使用 rpx 响应式单位
- 遵循 BEM 命名规范
- 使用 CSS 变量
- 避免深层嵌套（不超过 3 层）

---

## 参考资源

### 官方文档
- [微信官方文档 - 小程序框架](https://developers.weixin.qq.com/miniprogram/dev/framework/)
- [微信开发者工具文档](https://developers.weixin.qq.com/miniprogram/dev/devtools/devtools.html)
- [云开发文档](https://developers.weixin.qq.com/miniprogram/dev/wxcloud/)
- [性能优化指南](https://developers.weixin.qq.com/miniprogram/dev/framework/performance/)

### UI 组件库
- **WeUI**：追求微信原生视觉、快速搭建基础页面
- **TDesign**：企业级应用、多端统一、高定制需求
- **CloudBase Agent UI**：快速集成 AI 对话能力

### 开发者社区
- [小程序开发者社区](https://developers.weixin.qq.com/community/)

---

## 项目特点

### 相比其他类似项目的优势

| 维度 | 本 Skill 特点 |
|------|--------------|
| 场景定义 | ✅ 清晰的使用场景和关键词触发机制 |
| 代码规范 | ✅ 完整的命名、代码风格、组件化规范 |
| 性能优化 | ✅ 深入的 setData 优化、渲染优化、分包加载 |
| 防坑指南 | ✅ iOS/Android 兼容性问题解决方案 |
| 工具链 | ✅ 完整的开发工具链和调试技巧 |
| API 参考 | ✅ 全面的 API 速查手册 |
| 云开发 | ✅ 云函数、云数据库、云存储完整示例 |
| 测试策略 | ✅ 单元测试、自动化测试、真机调试 |
| 安全规范 | ✅ 数据加密、权限控制、XSS 防护 |
| UX 优化 | ✅ 骨架屏、加载反馈、错误处理 |

---

## 注意事项

1. **此项目不是可执行的代码项目**，而是文档和知识库集合
2. 主要用于指导 AI 助手在小程序开发场景中提供专业建议
3. 所有示例代码均为演示用途，实际使用时需根据具体需求调整
4. 项目中的 AppID 等配置信息需要替换为实际的值
