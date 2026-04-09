# 微信小程序错误处理与错误码

## 统一错误处理模板

```javascript
// utils/error-handler.js

/**
 * 错误码映射表
 */
const ERROR_CODES = {
  // 网络错误
  NETWORK_ERROR: { code: 1001, message: '网络连接失败，请检查网络' },
  TIMEOUT_ERROR: { code: 1002, message: '请求超时，请稍后重试' },

  // 授权错误
  AUTH_DENIED: { code: 2001, message: '用户拒绝授权' },
  AUTH_EXPIRED: { code: 2002, message: '登录已过期，请重新登录' },

  // 业务错误
  PARAM_INVALID: { code: 3001, message: '参数错误' },
  RESOURCE_NOT_FOUND: { code: 3002, message: '资源不存在' },
  OPERATION_FAILED: { code: 3003, message: '操作失败，请重试' },

  // 系统错误
  SYSTEM_ERROR: { code: 5001, message: '系统繁忙，请稍后重试' },
  UNKNOWN_ERROR: { code: 9999, message: '未知错误' }
}

/**
 * 统一错误处理类
 */
class ErrorHandler {
  /**
   * 处理错误
   * @param {Error|Object} error - 错误对象
   * @param {Object} options - 配置选项
   */
  static handle(error, options = {}) {
    const {
      showToast = true,
      logError = true,
      fallback = null
    } = options

    // 解析错误
    const errorInfo = this.parseError(error)

    // 记录日志
    if (logError) {
      console.error('[ErrorHandler]', errorInfo)

      // 上报错误（如有需要）
      this.reportError(errorInfo)
    }

    // 显示提示
    if (showToast) {
      wx.showToast({
        title: errorInfo.message,
        icon: 'none',
        duration: 2000
      })
    }

    // 返回降级数据
    return fallback
  }

  /**
   * 解析错误信息
   */
  static parseError(error) {
    // 如果是已定义的错误码
    if (error && error.code && ERROR_CODES[error.code]) {
      return {
        ...ERROR_CODES[error.code],
        originalError: error
      }
    }

    // 微信 API 错误
    if (error && error.errMsg) {
      return this.parseWxError(error)
    }

    // HTTP 错误
    if (error && error.statusCode) {
      return this.parseHttpError(error)
    }

    // 默认未知错误
    return {
      ...ERROR_CODES.UNKNOWN_ERROR,
      originalError: error
    }
  }

  /**
   * 解析微信 API 错误
   */
  static parseWxError(error) {
    const errMsg = error.errMsg || ''

    // 常见微信错误匹配
    if (errMsg.includes('fail auth deny') || errMsg.includes('fail authorize')) {
      return ERROR_CODES.AUTH_DENIED
    }
    if (errMsg.includes('fail timeout')) {
      return ERROR_CODES.TIMEOUT_ERROR
    }
    if (errMsg.includes('fail network')) {
      return ERROR_CODES.NETWORK_ERROR
    }

    return {
      code: 4000,
      message: errMsg.replace('request:fail ', '').replace('getUserProfile:fail ', ''),
      originalError: error
    }
  }

  /**
   * 解析 HTTP 错误
   */
  static parseHttpError(error) {
    const statusCode = error.statusCode

    const httpErrors = {
      400: ERROR_CODES.PARAM_INVALID,
      401: ERROR_CODES.AUTH_EXPIRED,
      404: ERROR_CODES.RESOURCE_NOT_FOUND,
      500: ERROR_CODES.SYSTEM_ERROR,
      502: ERROR_CODES.SYSTEM_ERROR,
      503: ERROR_CODES.SYSTEM_ERROR
    }

    return httpErrors[statusCode] || {
      code: 4000 + statusCode,
      message: `请求失败 (${statusCode})`,
      originalError: error
    }
  }

  /**
   * 上报错误（可选）
   */
  static reportError(errorInfo) {
    // 可以接入日志上报服务
    // 如 wx.getRealtimeLogManager 或第三方服务
    const logger = wx.getRealtimeLogManager?.()
    logger?.error('ErrorReport', errorInfo)
  }
}

// 便捷方法：包装异步函数
ErrorHandler.wrap = function(fn, options = {}) {
  return async function(...args) {
    try {
      return await fn.apply(this, args)
    } catch (error) {
      return ErrorHandler.handle(error, options)
    }
  }
}

module.exports = { ErrorHandler, ERROR_CODES }
```

## 使用示例

```javascript
const { ErrorHandler } = require('../utils/error-handler')

Page({
  // 方式1：使用 try-catch + ErrorHandler
  async fetchData() {
    try {
      const data = await request.get('/api/data')
      this.setData({ data })
    } catch (error) {
      ErrorHandler.handle(error, {
        showToast: true,
        fallback: { list: [] }
      })
    }
  },

  // 方式2：使用包装函数
  fetchUserInfo: ErrorHandler.wrap(async function() {
    const userInfo = await wx.getUserProfile({ desc: '用于完善资料' })
    this.setData({ userInfo })
  }, { fallback: null }),

  // 方式3：处理特定错误
  async login() {
    try {
      await this.doLogin()
    } catch (error) {
      if (error.code === 2002) {
        // 登录过期，重新登录
        await this.reLogin()
      } else {
        ErrorHandler.handle(error)
      }
    }
  }
})
```

## 微信小程序常见错误码对照表

### 全局错误码

| 错误码 | 说明 | 处理建议 |
|--------|------|---------|
| -1 | 系统错误 | 提示用户稍后重试 |
| 0 | 成功 | 正常处理 |

### 登录相关错误码

| 错误码 | 说明 | 处理建议 |
|--------|------|---------|
| 40001 | access_token 过期 | 重新获取 access_token |
| 40029 | code 无效 | 重新调用 wx.login 获取 code |
| 40163 | code 已被使用 | 重新调用 wx.login |
| 41008 | code 缺失 | 检查登录流程 |
| 42001 | access_token 超时 | 重新获取 access_token |
| 42002 | refresh_token 超时 | 重新登录 |

### 授权相关错误码

| 错误码 | 说明 | 处理建议 |
|--------|------|---------|
| 89019 | 用户拒绝授权 | 引导用户重新授权 |
| 50001 | 用户未授权该api | 引导用户开启权限 |

### 频率限制错误码

| 错误码 | 说明 | 处理建议 |
|--------|------|---------|
| 45009 | 调用频率限制 | 降低调用频率，添加防抖 |
| 45047 | 客服接口下行条数超过上限 | 减少推送频率 |

### 网络请求错误

| errMsg | 说明 | 处理建议 |
|--------|------|---------|
| request:fail timeout | 请求超时 | 检查网络，稍后重试 |
| request:fail ssl hand shake error | SSL 握手失败 | 检查服务器证书 |
| request:fail socket time out | Socket 超时 | 检查网络连接 |
| request:fail interrupted | 请求中断 | 重新发起请求 |
| request:fail -2:net::ERR_FAILED | 网络错误 | 检查网络连接 |

### 云开发错误码

| 错误码 | 说明 | 处理建议 |
|--------|------|---------|
| -401001 | SDK 未初始化 | 检查 wx.cloud.init 调用 |
| -402001 | 环境未找到 | 检查环境 ID 配置 |
| -403001 | 权限不足 | 检查数据库权限设置 |
| -404001 | 资源不存在 | 检查资源是否存在 |
| -501001 | 云函数调用失败 | 检查云函数部署状态 |
| -502001 | 云函数执行超时 | 优化云函数执行时间 |
| -503001 | 云函数内存不足 | 增加云函数内存配置 |

### 支付相关错误码

| 错误码 | 说明 | 处理建议 |
|--------|------|---------|
| 1000 | 参数格式错误 | 检查支付参数 |
| 1001 | 缺少参数 | 检查必填参数 |
| 1002 | 签名错误 | 检查签名算法 |
| 1003 | appid 错误 | 检查 appid 配置 |
| 1004 | 商户号错误 | 检查商户号配置 |

## 自定义错误码规范

建议按照以下规范定义业务错误码：

```
错误码格式：ABCD

A - 错误级别
  1: 网络/请求错误
  2: 授权/登录错误
  3: 业务逻辑错误
  4: 客户端错误
  5: 服务端错误

B - 模块分类
  0: 通用
  1: 用户模块
  2: 订单模块
  3: 支付模块
  4: 商品模块

CD - 具体错误序号
  01-99: 具体错误

示例：
1001 - 网络连接失败（通用网络错误）
2001 - 用户未登录（授权错误）
3001 - 订单不存在（订单业务错误）
3101 - 库存不足（订单业务错误-库存相关）
```

## 错误上报

```javascript
// utils/error-reporter.js

class ErrorReporter {
  constructor() {
    this.queue = []
    this.maxQueueSize = 10
    this.flushInterval = 30000 // 30秒上报一次

    // 定时上报
    setInterval(() => this.flush(), this.flushInterval)
  }

  report(error) {
    const errorInfo = {
      message: error.message,
      stack: error.stack,
      timestamp: Date.now(),
      page: getCurrentPages().pop()?.route || 'unknown',
      system: wx.getSystemInfoSync()
    }

    this.queue.push(errorInfo)

    // 队列满时立即上报
    if (this.queue.length >= this.maxQueueSize) {
      this.flush()
    }
  }

  async flush() {
    if (this.queue.length === 0) return

    const errors = [...this.queue]
    this.queue = []

    try {
      // 上报到服务器
      await wx.request({
        url: 'https://your-api.com/error-report',
        method: 'POST',
        data: { errors }
      })
    } catch (err) {
      // 上报失败，重新加入队列
      this.queue.unshift(...errors)
      // 防止无限增长
      if (this.queue.length > this.maxQueueSize * 2) {
        this.queue = this.queue.slice(0, this.maxQueueSize * 2)
      }
    }
  }
}

// 全局错误监听
const reporter = new ErrorReporter()

// 监听 JS 错误
wx.onError((error) => {
  reporter.report(new Error(error))
})

// 监听 Promise 未捕获错误
wx.onUnhandledRejection((res) => {
  reporter.report(new Error(res.reason))
})

module.exports = reporter
```
