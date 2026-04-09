# 微信小程序测试与调试指南

## 单元测试

使用 `miniprogram-simulate` 框架进行组件单元测试。

### 安装与配置

```bash
# 安装测试依赖
npm install --save-dev miniprogram-simulate jest
```

```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'node',
  moduleFileExtensions: ['js', 'json'],
  testMatch: ['**/__tests__/**/*.test.js'],
  collectCoverageFrom: [
    'components/**/*.js',
    'utils/**/*.js',
    '!**/node_modules/**'
  ]
}
```

### 组件测试示例

```javascript
// components/custom-button/__tests__/index.test.js
const simulate = require('miniprogram-simulate')

// 加载组件
const customButton = simulate.load('../index')

describe('CustomButton 组件', () => {
  it('应正确渲染按钮文本', () => {
    const comp = simulate.render(customButton, {
      title: '点击我'
    })

    comp.attach(document.createElement('parent-wrapper'))

    const btn = comp.querySelector('.custom-button')
    expect(btn.dom.textContent).toBe('点击我')
  })

  it('点击应触发 tap 事件', () => {
    const comp = simulate.render(customButton)
    comp.attach(document.createElement('parent-wrapper'))

    const tapHandler = jest.fn()
    comp.addEventListener('tap', tapHandler)

    const btn = comp.querySelector('.custom-button')
    btn.dispatchEvent('tap')

    expect(tapHandler).toHaveBeenCalled()
  })

  it('禁用状态应阻止点击', () => {
    const comp = simulate.render(customButton, {
      disabled: true
    })
    comp.attach(document.createElement('parent-wrapper'))

    const btn = comp.querySelector('.custom-button')
    expect(btn.dom.disabled).toBe(true)
  })
})
```

### 工具函数测试

```javascript
// utils/__tests__/util.test.js
const { formatTime, debounce } = require('../util')

describe('工具函数', () => {
  describe('formatTime', () => {
    it('应正确格式化日期', () => {
      const date = new Date('2024-03-15 14:30:00')
      expect(formatTime(date)).toBe('2024-03-15 14:30:00')
    })

    it('应处理时间戳', () => {
      const timestamp = 1710505800000
      expect(formatTime(timestamp)).toBe('2024-03-15 14:30:00')
    })
  })

  describe('debounce', () => {
    jest.useFakeTimers()

    it('应防抖执行函数', () => {
      const fn = jest.fn()
      const debouncedFn = debounce(fn, 300)

      debouncedFn()
      debouncedFn()
      debouncedFn()

      expect(fn).not.toHaveBeenCalled()

      jest.advanceTimersByTime(300)
      expect(fn).toHaveBeenCalledTimes(1)
    })
  })
})
```

## 自动化测试

使用微信开发者工具提供的自动化测试能力。

### 安装自动化 SDK

```bash
npm install miniprogram-automator
```

### 编写自动化测试脚本

```javascript
// auto-test/index.test.js
const automator = require('miniprogram-automator')

describe('小程序自动化测试', () => {
  let miniProgram
  let page

  beforeAll(async () => {
    miniProgram = await automator.launch({
      cliPath: 'C:\\Program Files (x86)\\Tencent\\微信web开发者工具\\cli.bat',
      projectPath: 'D:\\project\\miniprogram'
    })
  }, 30000)

  afterAll(async () => {
    await miniProgram.close()
  })

  beforeEach(async () => {
    page = await miniProgram.reLaunch('/pages/index/index')
    await page.waitFor(500)
  })

  it('首页应正确渲染', async () => {
    const title = await page.$('.page-title')
    expect(await title.text()).toBe('首页')

    const list = await page.$$('.list-item')
    expect(list.length).toBeGreaterThan(0)
  })

  it('点击按钮应跳转详情页', async () => {
    const btn = await page.$('.detail-btn')
    await btn.tap()

    await page.waitFor(500)

    const currentPage = await miniProgram.currentPage()
    expect(currentPage.path).toBe('pages/detail/detail')
  })

  it('表单提交应验证输入', async () => {
    const input = await page.$('input[name="username"]')
    await input.input('testuser')

    const submitBtn = await page.$('.submit-btn')
    await submitBtn.tap()

    const toast = await page.waitFor('.toast-message')
    expect(await toast.text()).toContain('提交成功')
  })
})
```

### 运行自动化测试

```bash
# 使用开发者工具 CLI
cli auto --project "D:\project\miniprogram" --auto-port 9420

# 或使用 Jest 运行
jest auto-test/
```

### 自动化测试 API 参考

| 方法 | 说明 |
|------|------|
| `automator.launch(options)` | 启动开发者工具 |
| `miniProgram.reLaunch(path)` | 重新启动到指定页面 |
| `miniProgram.navigateTo(path)` | 跳转到页面 |
| `miniProgram.navigateBack()` | 返回上一页 |
| `miniProgram.switchTab(path)` | 切换 Tab |
| `miniProgram.currentPage()` | 获取当前页面信息 |
| `page.$(selector)` | 查询单个元素 |
| `page.$$(selector)` | 查询多个元素 |
| `element.tap()` | 点击元素 |
| `element.input(text)` | 输入文本 |
| `element.text()` | 获取文本内容 |
| `element.attribute(name)` | 获取属性值 |
| `page.waitFor(selector)` | 等待元素出现 |
| `page.waitFor(timeout)` | 等待指定时间 |
| `page.callMethod(method, ...args)` | 调用页面方法 |
| `page.data()` | 获取页面数据 |

## 真机调试

### 开启真机调试

1. 微信开发者工具 → 真机调试 → 真机调试 2.0
2. 扫描二维码在手机上预览
3. 在电脑上查看调试信息

### 实时日志

```javascript
// 开启真机调试日志
const enableDebugLog = () => {
  // 使用实时日志上报
  const logger = wx.getRealtimeLogManager ? wx.getRealtimeLogManager() : null

  return {
    info(...args) {
      console.info(...args)
      logger?.info(...args)
    },
    warn(...args) {
      console.warn(...args)
      logger?.warn(...args)
    },
    error(...args) {
      console.error(...args)
      logger?.error(...args)
    }
  }
}

const log = enableDebugLog()

// 使用示例
log.info('页面加载完成', { page: 'index', userId: '123' })
log.error('请求失败', { url, error: err.message })
```

### 远程调试

```javascript
// 在真机调试时输出更多调试信息
const isDevTool = wx.getSystemInfoSync().platform === 'devtools'

const debug = {
  log(...args) {
    if (isDevTool) {
      console.log('[Debug]', ...args)
    }
  },
  table(data) {
    if (isDevTool) {
      console.table(data)
    }
  }
}
```

## 性能测试

### 性能监控

```javascript
// 性能监控
const monitorPerformance = () => {
  if (!wx.getPerformance) return

  const performance = wx.getPerformance()
  const observer = performance.createObserver((entryList) => {
    const entries = entryList.getEntries()

    entries.forEach(entry => {
      // 上报性能数据
      console.log('性能指标:', {
        name: entry.name,
        duration: entry.duration,
        entryType: entry.entryType
      })

      // 警告慢渲染
      if (entry.duration > 100) {
        console.warn('渲染耗时过长:', entry.name, entry.duration)
      }
    })
  })

  observer.observe({ entryTypes: ['render', 'script'] })
}

// 在 app.js 中调用
App({
  onLaunch() {
    monitorPerformance()
  }
})
```

### 内存监控

```javascript
// 内存监控（仅在开发者工具中有效）
const monitorMemory = () => {
  setInterval(() => {
    if (wx.getPerformance) {
      const memory = wx.getPerformance().memory
      if (memory) {
        console.log('内存使用:', {
          usedJSHeapSize: (memory.usedJSHeapSize / 1048576).toFixed(2) + ' MB',
          totalJSHeapSize: (memory.totalJSHeapSize / 1048576).toFixed(2) + ' MB'
        })
      }
    }
  }, 5000)
}
```

## 调试技巧

### 使用 vConsole

```javascript
// 在开发环境下启用 vConsole
if (wx.getAccountInfoSync().miniProgram.envVersion === 'develop') {
  // 开发版自动开启调试
  wx.setEnableDebug({
    enableDebug: true
  })
}
```

### 网络请求监控

```javascript
// 封装请求以统一监控
const originalRequest = wx.request
wx.request = function(options) {
  const startTime = Date.now()

  const originalSuccess = options.success
  options.success = function(res) {
    console.log(`[Request] ${options.url}`, {
      status: res.statusCode,
      duration: Date.now() - startTime + 'ms',
      data: res.data
    })
    originalSuccess && originalSuccess(res)
  }

  const originalFail = options.fail
  options.fail = function(err) {
    console.error(`[Request Failed] ${options.url}`, err)
    originalFail && originalFail(err)
  }

  return originalRequest(options)
}
```

### 页面生命周期监控

```javascript
// 监控页面生命周期
const originalPage = Page
Page = function(config) {
  const lifecycles = ['onLoad', 'onShow', 'onReady', 'onHide', 'onUnload']

  lifecycles.forEach(lifecycle => {
    const original = config[lifecycle]
    config[lifecycle] = function(...args) {
      console.log(`[Page Lifecycle] ${this.route} - ${lifecycle}`)
      return original && original.apply(this, args)
    }
  })

  return originalPage(config)
}
```
