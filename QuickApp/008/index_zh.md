# 第8篇：常用工具篇（下）

在鸿蒙应用开发中，工具类的合理封装和复用是提升开发效率和代码健壮性的关键。本篇继续梳理项目中常用的实用工具，包括权限管理、日志打印、窗口与屏幕管理、系统API封装、日期格式化、音频处理等，所有API和术语均严格参考[华为开发者官网](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/development-intro-api)。

## 一、权限申请 —— permission

权限管理是保障应用安全和合规的基础。推荐使用`@kit.AbilityKit`的`abilityAccessCtrl`能力进行权限检测和动态申请。
- 检查权限：`await permission.checkPermission(Permissions.PERMISSION_NAME)`
- 申请权限：`await permission.requestPermission(Permissions.PERMISSION_NAME)`
- 回调方式：`permission.request(Permissions.PERMISSION_NAME, (agree) => { ... })`
- 权限需在`module.json5`声明。
- 相关API：[权限管理](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/app-permission-mgmt)

## 二、日志打印 —— log

日志是开发和运维的重要工具。推荐基于`@ohos.hilog`封装日志类，支持多级别（debug/info/warn/error/fatal）、动态关闭、设置打印等级等。
- 典型用法：`log.d(TAG, '调试信息')`
- 关闭日志：`log.closeLog()`
- 设置等级：`log.setLevel(hilog.LogLevel.INFO)`
- 相关API：[日志打印](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/hilog-guidelines-arkts)

## 三、窗口与屏幕管理 —— WindowUtil

用于监听和管理状态栏、导航栏、屏幕宽高变化，支持全屏、暗黑模式、系统栏显示隐藏等。
- 监听变化：`WindowUtil.listenToAppStorage(windowClass)`
- 设置暗黑模式：`WindowUtil.setDartEnable(windowClass, true)`
- 设置全屏：`WindowUtil.setFullScreenHideBar(windowClass)`
- 相关API：[窗口管理](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-window-stage)

## 四、系统API封装 —— sysUtil

以剪切板为例，封装常用系统API，提升开发效率。
- 复制文本：`sysUtil.copy('内容')`
- 相关API：[剪切板](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-pasteboard)

## 五、日期时间格式化 —— DateUtil

用于格式化时间戳、Date对象为常用字符串格式，支持自定义模板。
- 格式化：`DateUtil.formatDate(Date.now(), 'YYYY-MM-DD HH:mm:ss')`
- 相关API：[日期时间处理](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-intl)

## 六、PCM转WAV音频处理 —— wav

用于将PCM音频数据封装为WAV格式并保存到本地，适合录音等场景。
- 构造写入对象：`let wavWriter = new wav.Writer(...)`
- 写入PCM数据：`wavWriter.writeData(buffer)`
- 结束写入：`wavWriter.writeFinish()`
- 相关API：[文件操作](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/core-file-kit-intro)

## 七、最佳实践与官方文档

- 工具类建议单一职责、接口简洁，便于复用和维护。
- 重要能力如权限、日志、窗口管理等务必参考[官方文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-coding-style-guide)和[ArkTS编程规范](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-coding-style-guide-V5)。
- 代码应有详细注释，关键异常需日志记录，便于排查。

**到这里，所有的内容已经结束了！本章的完整源码已经上传到gitee了：[鸿蒙应用0-1开发](https://gitee.com/qincji/ZeroOneApp)。**