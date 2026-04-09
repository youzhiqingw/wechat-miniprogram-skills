# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a WeChat Mini Program development skills package (微信小程序开发技能库) designed for AI programming assistants. It provides comprehensive development guidance, performance optimization tips, and best practices for WeChat Mini Program development.

The repository contains:
- `SKILL.md` - Main skill definition file with complete development guidelines
- `README.md` - Project overview and quick start guide
- `references/` - Detailed reference documentation
- `examples/` - Code examples and templates
- `assets/` - Static resources (images)

## Key Files Structure

```
wechat-miniprogram-skills/
├── SKILL.md                          # Main skill definition (core file)
├── README.md                         # Project overview
├── QWEN.md                           # Qwen model adaptation
├── references/
│   ├── api-reference.md              # API quick reference
│   ├── performance-optimization.md   # Performance optimization guide
│   ├── version-compatibility.md      # Base library compatibility
│   ├── testing-guide.md              # Testing and debugging
│   ├── error-handling.md             # Error handling patterns
│   ├── deployment-guide.md           # Release and deployment
│   └── ui-components-guide.md        # UI component library selection
└── examples/
    ├── basic-template.md             # Basic project template
    ├── cloud-development.md          # Cloud development example
    ├── typescript-template.md        # TypeScript project template
    └── basic-project/
        └── README.md                 # Basic project guide
```

## When to Use This Skill

This skill should be activated when users mention:
- WeChat Mini Program (微信小程序) development
- WXML, WXSS, WXS files or syntax
- `wx.*` APIs (e.g., `wx.request`, `wx.login`, `wx.navigateTo`)
- Mini Program components, pages, cloud functions
- AppID, `project.config.json`, `app.json`
- WeChat Developer Tools, preview, upload, publish
- setData optimization, page stack, subpackage loading
- iOS/Android compatibility issues in Mini Programs

## Core Principles (from SKILL.md)

When assisting with WeChat Mini Program development, always follow these 5 principles:

1. **Performance First**: Optimize setData calls (use data paths for partial updates), reduce rendering overhead, implement subpackage loading
2. **Native Compatibility**: Handle iOS/Android differences (date formats, keyboard handling, etc.)
3. **Code Quality**: Modular, maintainable code with proper error handling
4. **User Experience**: Fast response, loading indicators, skeleton screens
5. **Security**: No hardcoded secrets, encrypt sensitive data, prevent XSS

## Critical Technical Details

### setData Optimization (Most Important)
```javascript
// ✅ Use data paths for partial updates
this.setData({
  'userInfo.nickName': 'New Name',
  'list[0].status': 'completed'
})

// ❌ Avoid full object updates
this.setData({
  userInfo: { ...this.data.userInfo, nickName: 'New Name' }
})
```

### iOS Date Format Compatibility
```javascript
// ❌ iOS doesn't support hyphen format
new Date('2024-03-31') // Returns Invalid Date on iOS

// ✅ Use slash format for compatibility
new Date('2024/03/31')
```

### Page Stack Limit (Max 10)
```javascript
const pages = getCurrentPages()
if (pages.length >= 10) {
  wx.redirectTo({ url: '/pages/target/target' })
} else {
  wx.navigateTo({ url: '/pages/target/target' })
}
```

### Native Component Layering
Native components (`video`, `map`, `canvas`, `camera`, `live-player`) are always on top. Use `cover-view` and `cover-image` to overlay them:
```xml
<video src="{{videoUrl}}">
  <cover-view class="controls">Overlay content</cover-view>
</video>
```

## Package Size Limits

- Main package: < 2MB
- Total package: < 20MB
- Single subpackage: < 2MB
- setData payload: < 1024KB per call

## Reference Documentation

When users need detailed information, refer them to:
- API details: `references/api-reference.md`
- Performance: `references/performance-optimization.md`
- Testing: `references/testing-guide.md`
- Deployment: `references/deployment-guide.md`
- UI Components: `references/ui-components-guide.md`

## Examples

- Basic template: `examples/basic-template.md`
- Cloud development: `examples/cloud-development.md`
- TypeScript: `examples/typescript-template.md`
