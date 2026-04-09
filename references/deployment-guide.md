# 微信小程序发布与部署指南

## 备案指引（2023年9月起强制）

根据工信部要求，**自2023年9月1日起，所有新上线的小程序必须完成备案才能发布**。未备案的小程序将无法通过审核，已上线的小程序需在2024年3月31日前完成备案。

### 备案流程

```
1. 准备材料
   ├── 主办单位证件（企业营业执照/个人身份证）
   ├── 主体负责人身份证
   ├── 小程序负责人身份证（可与主体负责人相同）
   ├── 前置审批文件（如涉及新闻、教育、医疗等）
   └── 承诺书（在线签署）

2. 登录微信公众平台
   └── 设置 → 小程序备案 → 开始备案

3. 填写备案信息
   ├── 主体信息（与营业执照一致）
   ├── 主体负责人信息
   ├── 小程序信息（名称、类目、简介）
   └── 小程序负责人信息

4. 平台初审（1-3个工作日）
   └── 腾讯审核人员电话核实

5. 管局审核（约20个工作日）
   └── 各地通信管理局审核

6. 备案成功
   └── 获得备案号，可正常发布
```

### 备案注意事项

- **备案期间**：小程序可正常开发和测试，但无法提交审核
- **备案号展示**：需在小程序显著位置展示备案号（通常放在"关于我们"或设置页）
- **信息变更**：主体信息、负责人信息变更需重新备案
- **跨省备案**：服务器所在地与主体所在地不一致需特别注意

### 备案状态查询

```javascript
// 在小程序中展示备案信息
Page({
  data: {
    recordInfo: {
      number: '京ICP备XXXXXXXX号-1X',
      url: 'https://beian.miit.gov.cn/'
    }
  },

  // 点击备案号跳转
  onTapRecord() {
    wx.navigateTo({
      url: '/pages/webview/webview?url=https://beian.miit.gov.cn/'
    })
  }
})
```

---

## 用户隐私保护指引（2024年起强制）

**自2024年起，小程序必须在上线前配置隐私保护指引，否则部分涉及用户隐私的 API 将无法使用。**

### 需要配置隐私指引的场景

| 功能 | 涉及 API | 隐私类型 |
|------|---------|---------|
| 获取用户信息 | `wx.getUserProfile`, `wx.getUserInfo` | 用户头像、昵称 |
| 获取手机号 | `wx.getPhoneNumber` | 用户手机号 |
| 获取位置 | `wx.getLocation`, `wx.chooseLocation` | 地理位置 |
| 选择通讯录 | `wx.chooseContact` | 通讯录信息 |
| 选择图片/视频 | `wx.chooseImage`, `wx.chooseMedia` | 相册内容 |
| 摄像头 | `wx.scanCode`, `camera` 组件 | 拍摄内容 |
| 剪切板 | `wx.getClipboardData` | 剪切板内容 |

### 配置步骤

1. **登录微信公众平台** → 设置 → 基本设置 → 用户隐私保护指引

2. **填写隐私政策**（参考模板）：

```
本小程序尊重并保护所有使用服务用户的个人隐私权。

1. 信息收集
   我们会收集以下信息：
   - 用户信息：用于完善个人资料
   - 位置信息：用于提供附近的服务
   - 手机号：用于身份验证和联系

2. 信息使用
   我们收集的信息将用于：
   - 提供和改进服务
   - 用户身份验证
   - 与您联系

3. 信息保护
   我们使用各种安全技术和程序保护您的个人信息。

4. 信息共享
   我们不会向第三方共享您的个人信息，除非：
   - 获得您的明确同意
   - 法律法规要求
```

3. **配置 API 隐私说明**：

在 `app.json` 或页面 `json` 中配置：

```json
{
  "permission": {
    "scope.userLocation": {
      "desc": "你的位置信息将用于小程序位置接口的效果展示"
    },
    "scope.camera": {
      "desc": "你的摄像头将用于扫码和拍照功能"
    }
  },
  "requiredPrivateInfos": [
    "getLocation",
    "chooseLocation"
  ]
}
```

### 隐私协议弹窗实现

```javascript
// components/privacy-popup/privacy-popup.js
Component({
  data: {
    showPrivacy: false,
    privacyContractName: '',
    resolvePrivacyAuthorization: null
  },

  lifetimes: {
    attached() {
      // 监听隐私协议需要同意事件
      if (wx.onNeedPrivacyAuthorization) {
        wx.onNeedPrivacyAuthorization((resolve) => {
          this.setData({
            showPrivacy: true,
            resolvePrivacyAuthorization: resolve
          })
        })
      }

      // 获取隐私协议名称
      this.getPrivacySetting()
    }
  },

  methods: {
    // 获取隐私设置
    getPrivacySetting() {
      if (wx.getPrivacySetting) {
        wx.getPrivacySetting({
          success: (res) => {
            this.setData({
              privacyContractName: res.privacyContractName
            })
          }
        })
      }
    },

    // 同意隐私协议
    handleAgree() {
      this.data.resolvePrivacyAuthorization({
        buttonId: 'agree-btn',
        event: 'agree'
      })
      this.setData({ showPrivacy: false })
    },

    // 拒绝隐私协议
    handleDisagree() {
      this.data.resolvePrivacyAuthorization({
        event: 'disagree'
      })
      this.setData({ showPrivacy: false })
    },

    // 打开隐私协议
    openPrivacyContract() {
      wx.openPrivacyContract({
        success: () => {
          console.log('打开隐私协议成功')
        }
      })
    }
  }
})
```

```xml
<!-- components/privacy-popup/privacy-popup.wxml -->
<view class="privacy-popup" wx:if="{{showPrivacy}}">
  <view class="privacy-mask"></view>
  <view class="privacy-content">
    <view class="privacy-title">隐私保护指引</view>
    <view class="privacy-desc">
      使用本小程序需要同意
      <text class="privacy-link" bindtap="openPrivacyContract">
        《{{privacyContractName}}》
      </text>
    </view>
    <view class="privacy-actions">
      <button class="btn-disagree" bindtap="handleDisagree">拒绝</button>
      <button
        class="btn-agree"
        id="agree-btn"
        open-type="agreePrivacyAuthorization"
        bindagreeprivacyauthorization="handleAgree"
      >
        同意
      </button>
    </view>
  </view>
</view>
```

---

## 域名配置

### 服务器域名配置

登录微信公众平台 → 开发管理 → 开发设置 → 服务器域名

| 域名类型 | 用途 | 数量限制 | 修改频率 |
|---------|------|---------|---------|
| request 合法域名 | HTTPS 请求 | 200个 | 每月5次 |
| socket 合法域名 | WebSocket | 200个 | 每月5次 |
| uploadFile 合法域名 | 文件上传 | 200个 | 每月5次 |
| downloadFile 合法域名 | 文件下载 | 200个 | 每月5次 |
| UDP 合法域名 | UDP 通信 | 200个 | 每月5次 |
| TCP 合法域名 | TCP 通信 | 200个 | 每月5次 |

### 域名配置要求

1. **必须使用 HTTPS**
   - TLS 版本必须 ≥ 1.2
   - 证书必须有效且未过期
   - 不支持 IP 地址和 localhost

2. **ICP 备案**
   - 所有域名必须完成 ICP 备案
   - 境外域名无法配置

3. **业务域名**（web-view）
   - 用于 `web-view` 组件
   - 需下载校验文件上传到域名根目录
   - 修改无次数限制

### 开发环境跳过域名检查

```javascript
// 开发版/体验版可在开发者工具中开启
// 开发者工具 → 详情 → 本地设置 → 不校验合法域名

// 注意：生产环境必须配置合法域名
```

---

## 发布流程

### 发布前检查清单

```markdown
□ 代码审核
  □ 无 console.log 调试代码
  □ 无 alert 等浏览器 API
  □ 无硬编码敏感信息（AppID、密钥等）
  □ 代码包体积 < 2MB（主包）

□ 功能测试
  □ 所有页面可正常访问
  □ 核心功能流程完整
  □ 网络异常处理完善
  □ 授权流程正常

□ 合规检查
  □ 已完成备案（如需要）
  □ 已配置隐私保护指引
  □ 用户协议和隐私政策完整
  □ 无违规内容（黄赌毒、欺诈等）

□ 体验优化
  □ 首屏加载时间 < 2s
  □ 有加载状态提示
  □ 错误提示友好
  □ 适配 iOS/Android 双端

□ 配置检查
  □ 服务器域名已配置
  □ 业务域名已配置（如有 web-view）
  □ 白名单已配置（如有需要）
  □ AppID 正确
```

### 版本发布流程

```
1. 上传代码
   └── 微信开发者工具 → 上传 → 填写版本号、项目备注

2. 提交审核
   └── 微信公众平台 → 管理 → 版本管理 → 提交审核
   └── 填写功能页面、测试账号、备注说明

3. 等待审核
   └── 普通审核：1-3个工作日
   └── 加急审核：2小时内（每月有限额）

4. 审核结果
   ├── 审核通过 → 可发布
   └── 审核驳回 → 修改后重新提交

5. 发布上线
   └── 全量发布 或 灰度发布（分阶段放量）
```

### 版本管理

```javascript
// 获取小程序版本信息
const accountInfo = wx.getAccountInfoSync()
console.log(accountInfo.miniProgram)
// {
//   appId: "wxe5f52902cf4de896",
//   envVersion: "develop", // develop(开发版) | trial(体验版) | release(正式版)
//   version: "1.2.3"       // 线上版本号
// }

// 根据版本执行不同逻辑
if (accountInfo.miniProgram.envVersion === 'develop') {
  // 开发版开启调试
  wx.setEnableDebug({ enableDebug: true })
}
```

---

## CI/CD 自动化部署

### 使用 miniprogram-ci

```bash
# 安装 CI 工具
npm install miniprogram-ci
```

```javascript
// ci/upload.js
const ci = require('miniprogram-ci')

;(async () => {
  const project = new ci.Project({
    appid: 'your-appid',
    type: 'miniProgram',
    projectPath: './dist',
    privateKeyPath: './private.key', // 下载的私钥
    ignores: ['node_modules/**/*']
  })

  // 上传代码
  const uploadResult = await ci.upload({
    project,
    version: '1.2.3',
    desc: '修复已知问题，优化用户体验',
    setting: {
      es6: true,
      minify: true,
      autoPrefixWXSS: true
    }
  })

  console.log(uploadResult)
})()
```

### GitHub Actions 配置

```yaml
# .github/workflows/deploy.yml
name: Deploy MiniProgram

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Build
        run: npm run build

      - name: Upload MiniProgram
        run: node ci/upload.js
        env:
          MINI_APPID: ${{ secrets.MINI_APPID }}
          MINI_PRIVATE_KEY: ${{ secrets.MINI_PRIVATE_KEY }}
```

---

## 常见问题

### 审核驳回常见原因

| 原因 | 解决方案 |
|------|---------|
| 功能不完整 | 确保核心功能可用，不要提交测试版本 |
| 内容违规 | 检查是否有敏感词、违规图片 |
| 诱导分享 | 去除强制分享、诱导分享逻辑 |
| 未配置隐私指引 | 在后台配置隐私保护指引 |
| 未备案 | 完成备案后再提交 |
| 类目不符 | 选择与功能匹配的服务类目 |

### 域名配置问题

```
问题：request:fail url not in domain list
解决：检查域名是否已配置，注意区分 http/https

问题：request:fail ssl hand shake error
解决：检查服务器 SSL 证书配置，确保 TLS 1.2+

问题：web-view 无法加载
解决：配置业务域名并上传校验文件到根目录
```

### 发布限制

- 每个账号同时只能有一个审核中的版本
- 一天最多提交 5 次审核
- 加急审核每月有限额（一般 3 次）
- 代码包大小限制：2MB（主包）、4MB（总包）、20MB（含分包）
