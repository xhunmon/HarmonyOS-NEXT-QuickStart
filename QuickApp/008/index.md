# Part 8: Common Utility Tools (Part 2)

In HarmonyOS app development, proper encapsulation and reuse of utility classes are key to improving development efficiency and code robustness. This article continues to summarize practical utility tools commonly used in projects, including permission management, logging, window and screen management, system API encapsulation, date formatting, and audio processing. All APIs and terminology strictly follow the [Huawei Developer Official Documentation](https://developer.huawei.com/consumer/en/doc/harmonyos-references/development-intro-api).

## 1. Permission Management — permission

Permission management is fundamental to ensuring application security and compliance. It is recommended to use the `abilityAccessCtrl` capability from `@kit.AbilityKit` for permission checking and dynamic requests.
- Check permission: `await permission.checkPermission(Permissions.PERMISSION_NAME)`
- Request permission: `await permission.requestPermission(Permissions.PERMISSION_NAME)`
- Callback style: `permission.request(Permissions.PERMISSION_NAME, (agree) => { ... })`
- Permissions must be declared in `module.json5`.
- Related API: [Permission Management](https://developer.huawei.com/consumer/en/doc/harmonyos-guides/app-permission-mgmt)

## 2. Logging — log

Logging is an important tool for development and operations. It is recommended to encapsulate the log class based on `@ohos.hilog`, supporting multiple levels (debug/info/warn/error/fatal), dynamic disabling, and log level settings.
- Typical usage: `log.d(TAG, 'Debug info')`
- Disable logging: `log.closeLog()`
- Set log level: `log.setLevel(hilog.LogLevel.INFO)`
- Related API: [Logging Guidelines](https://developer.huawei.com/consumer/en/doc/harmonyos-guides/hilog-guidelines-arkts)

## 3. Window and Screen Management — WindowUtil

Used to listen for and manage changes in the status bar, navigation bar, and screen dimensions. Supports full screen, dark mode, and system bar show/hide.
- Listen for changes: `WindowUtil.listenToAppStorage(windowClass)`
- Set dark mode: `WindowUtil.setDartEnable(windowClass, true)`
- Set full screen: `WindowUtil.setFullScreenHideBar(windowClass)`
- Related API: [Window Management](https://developer.huawei.com/consumer/en/doc/harmonyos-guides/application-window-stage)

## 4. System API Encapsulation — sysUtil

Taking the clipboard as an example, encapsulate common system APIs to improve development efficiency.
- Copy text: `sysUtil.copy('content')`
- Related API: [Clipboard](https://developer.huawei.com/consumer/en/doc/harmonyos-references/js-apis-pasteboard)

## 5. Date and Time Formatting — DateUtil

Used to format timestamps or Date objects into common string formats, supporting custom templates.
- Format: `DateUtil.formatDate(Date.now(), 'YYYY-MM-DD HH:mm:ss')`
- Related API: [Date and Time Handling](https://developer.huawei.com/consumer/en/doc/harmonyos-references/js-apis-intl)

## 6. PCM to WAV Audio Processing — wav

Used to encapsulate PCM audio data into WAV format and save it locally, suitable for recording scenarios.
- Construct writer object: `let wavWriter = new wav.Writer(...)`
- Write PCM data: `wavWriter.writeData(buffer)`
- Finish writing: `wavWriter.writeFinish()`
- Related API: [File Operations](https://developer.huawei.com/consumer/en/doc/harmonyos-guides/core-file-kit-intro)

## 7. Best Practices and Official Documentation

- Utility classes should follow the single responsibility principle and have simple interfaces for easy reuse and maintenance.
- For important capabilities such as permissions, logging, and window management, always refer to the [Official Documentation](https://developer.huawei.com/consumer/en/doc/harmonyos-guides/arkts-coding-style-guide) and [ArkTS Coding Style Guide](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-coding-style-guide-V5).
- Code should include detailed comments, and key exceptions should be logged for troubleshooting.

**All content for this chapter is complete! The full source code has been uploaded to Gitee: [HarmonyOS App 0-1 Development](https://gitee.com/qincji/ZeroOneApp).** 