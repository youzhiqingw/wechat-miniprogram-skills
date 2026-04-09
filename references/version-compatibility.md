# 微信小程序基础库版本兼容性指南

微信小程序基础库版本决定了可用 API 的范围。以下是关键 API 的最低版本要求：

## 常用 API 版本要求

| API | 最低基础库版本 | 说明 |
|-----|---------------|------|
| `wx.getUserProfile` | 2.10.4 | 获取用户信息，需用户点击触发 |
| `wx.getUserInfo` | 1.3.0 | 旧版获取用户信息（已逐步回收） |
| `wx.cloud` | 2.2.3 | 云开发能力 |
| `wx.getRealtimeLogManager` | 2.7.1 | 实时日志上报 |
| `wx.getPerformance` | 2.11.0 | 性能监控 |
| `wx.onKeyboardHeightChange` | 2.7.0 | 监听键盘高度变化 |
| `wx.getGroupEnterInfo` | 2.7.13 | 获取群聊入口信息 |
| `wx.requestSubscribeMessage` | 2.4.4 | 订阅消息授权 |
| `wx.openCustomerServiceChat` | 2.12.0 | 打开客服会话 |
| `wx.getPhoneNumber` | 1.0.0 | 获取手机号（需后端解密） |
| `wx.login` | 1.0.0 | 微信登录 |
| `wx.request` | 1.0.0 | 网络请求 |
| `wx.getSystemInfo` | 1.0.0 | 获取系统信息 |
| `wx.getSetting` | 1.2.0 | 获取授权设置 |
| `wx.authorize` | 1.2.0 | 提前授权 |
| `wx.chooseImage` | 1.0.0 | 选择图片 |
| `wx.chooseMedia` | 2.10.0 | 选择媒体文件 |
| `wx.createVideoContext` | 1.6.0 | 视频上下文 |
| `wx.createCanvasContext` | 1.0.0 | Canvas 上下文 |
| `wx.getLocation` | 1.0.0 | 获取位置 |
| `wx.openLocation` | 1.0.0 | 打开位置 |
| `wx.scanCode` | 1.0.0 | 扫码 |
| `wx.showShareMenu` | 1.4.0 | 显示分享按钮 |
| `wx.updateShareMenu` | 1.4.0 | 更新分享菜单 |
| `wx.getShareInfo` | 1.4.0 | 获取分享信息 |
| `wx.navigateToMiniProgram` | 1.3.0 | 跳转到其他小程序 |
| `wx.navigateBackMiniProgram` | 1.3.0 | 返回到上一个小程序 |
| `wx.startPullDownRefresh` | 1.5.0 | 开始下拉刷新 |
| `wx.pageScrollTo` | 1.4.0 | 页面滚动 |
| `wx.createSelectorQuery` | 1.4.0 | 节点查询 |
| `wx.createIntersectionObserver` | 1.6.0 | 节点交叉观察 |
| `wx.getMenuButtonBoundingClientRect` | 2.1.0 | 获取胶囊按钮位置 |
| `wx.setTabBarBadge` | 1.9.0 | 设置 TabBar 徽标 |
| `wx.showTabBarRedDot` | 1.9.0 | 显示 TabBar 红点 |
| `wx.setBackgroundColor` | 2.1.0 | 设置背景色 |
| `wx.setBackgroundTextStyle` | 2.1.0 | 设置背景文字样式 |

## 版本兼容性检查

```javascript
// 检查基础库版本
const checkVersion = (requiredVersion) => {
  const systemInfo = wx.getSystemInfoSync()
  const currentVersion = systemInfo.SDKVersion

  // 版本号比较
  const compareVersion = (v1, v2) => {
    const v1Arr = v1.split('.').map(Number)
    const v2Arr = v2.split('.').map(Number)

    for (let i = 0; i < 3; i++) {
      if (v1Arr[i] > v2Arr[i]) return 1
      if (v1Arr[i] < v2Arr[i]) return -1
    }
    return 0
  }

  return compareVersion(currentVersion, requiredVersion) >= 0
}

// 使用示例
if (checkVersion('2.10.4')) {
  // 可以使用 wx.getUserProfile
  this.getUserProfile()
} else {
  // 降级处理
  wx.showModal({
    title: '提示',
    content: '当前微信版本过低，请升级后使用',
    showCancel: false
  })
}
```

## 兼容性处理最佳实践

```javascript
// ✅ 推荐：使用 wx.canIUse 检查 API 可用性
if (wx.canIUse('getUserProfile')) {
  // 使用新版 API
  const { userInfo } = await wx.getUserProfile({ desc: '用于完善资料' })
} else if (wx.canIUse('getUserInfo')) {
  // 降级使用旧版 API
  const { userInfo } = await wx.getUserInfo()
} else {
  // 提示用户升级微信
  wx.showModal({
    title: '提示',
    content: '请升级微信版本以使用完整功能'
  })
}

// ✅ 推荐：使用 try-catch 处理可能的错误
try {
  const result = await wx.someNewAPI()
} catch (err) {
  if (err.errMsg && err.errMsg.includes('not support')) {
    // API 不支持，使用降级方案
    const result = await fallbackMethod()
  }
}
```

## 基础库版本设置

在 `project.config.json` 中设置最低基础库版本：

```json
{
  "setting": {
    "urlCheck": true,
    "es6": true,
    "enhance": true,
    "postcss": true,
    "preloadBackgroundData": false,
    "minified": true,
    "newFeature": false,
    "coverView": true,
    "nodeModules": false,
    "autoAudits": false,
    "showShadowRootInWxmlPanel": true,
    "scopeDataCheck": false,
    "uglifyFileName": false,
    "checkInvalidKey": true,
    "checkSiteMap": true,
    "uploadWithSourceMap": true,
    "compileHotReLoad": false,
    "lazyloadPlaceholderEnable": false,
    "useMultiFrameRuntime": true,
    "useApiHook": true,
    "useApiHostProcess": true,
    "babelSetting": {
      "ignore": [],
      "disablePlugins": [],
      "outputPath": ""
    },
    "enableEngineNative": false,
    "useIsolateContext": true,
    "userConfirmedBundleSwitch": false,
    "packNpmManually": false,
    "packNpmRelationList": [],
    "minifyWXSS": true,
    "disableUseStrict": false,
    "minifyWXML": true,
    "showES6CompileOption": false,
    "useCompilerPlugins": false
  },
  "libVersion": "2.19.4"
}
```

## 版本分布参考

根据微信官方数据，截至 2024 年：

| 基础库版本 | 用户覆盖率 |
|-----------|-----------|
| 2.20.0+ | 95% |
| 2.15.0+ | 98% |
| 2.10.0+ | 99.5% |
| 2.5.0+ | 99.9% |

建议将最低支持版本设置为 2.10.0，以覆盖绝大多数用户。
