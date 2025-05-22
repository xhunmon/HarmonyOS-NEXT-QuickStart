# Part 7: Common Utility Tools (Part 1)

In HarmonyOS app development, properly encapsulating and using utility classes can greatly improve development efficiency and code quality. This article systematically summarizes the commonly used utility classes in the project, including device unique ID generation, global object caching, page navigation, and data persistence. All APIs and terminology strictly follow the [Huawei Developer Official Documentation](https://developer.huawei.com/consumer/en/doc/harmonyos-references/development-intro-api).

## 1. Device Unique ID Generation — DeviceUtil

A unique device identifier is crucial for user behavior analysis and data isolation. It is recommended to use the `@ohos.security.asset` capability to generate a persistent ID (which does not change after uninstall/reinstall). If unavailable, fallback to AAID.

- Permission configuration:
```json
{
  "name": "ohos.permission.STORE_PERSISTENT_DATA",
  ...
}
```
- Main implementation ideas:
  1. Use `asset.AssetStoreKit` to query or generate a unique ID and persist it.
  2. If not supported, use `AAID.getAAID()`, but note that it changes after uninstalling or factory resetting the device.
- Typical usage:
```ts
const deviceId = await DeviceUtil.getDeviceID();
```

## 2. Global Object Caching — global

The global object caching utility class can be used to store objects such as UI context and functions, making cross-module calls easier. It can also be used to prevent rapid repeated button clicks.

- Typical usage:
```ts
global.setObject('UI_CONTEXT', getContext(this));
const uiContext = global.getObject<common.UIAbilityContext>('UI_CONTEXT');
Button('Login').onClick(global.blockQuickClick(() => { /* login logic */ }))
```
- Related API: [Context Management](https://developer.huawei.com/consumer/en/doc/harmonyos-guides/application-context-stage)

## 3. Page Navigation Utility — IntentUtil

Page navigation and third-party app invocation are common needs in mobile development. The IntentUtil utility class supports:
- Navigate to WebView page: `IntentUtil.jumpWeb(url, title)`
- Navigate to AppGallery: `IntentUtil.jumpStore(appId)`
- Navigate to another app: `IntentUtil.jumpApp(deviceId, bundleName, abilityName)`
- Implicit navigation (e.g., open a document): `IntentUtil.jumpAction(action, entities, uri, type)`
- Smart parsing and navigation: `IntentUtil.jumpUrl(url or json)`

- Related API: [AbilityKit Inter-Device Interaction](https://developer.huawei.com/consumer/en/doc/harmonyos-guides/uiability-intra-device-interaction)

## 4. Data Persistence (Below 4MB) — kvManger

Suitable for storing larger data (such as base64 images), implemented based on `distributedKVStore`, and supports both local and multi-device collaboration.
- Initialization: `kvManger.init(this.context)`
- Put/Get/Delete: `put(key, value)`, `get(key, defVal)`, `delete(key)`
- Related API: [Distributed KV Store](https://developer.huawei.com/consumer/en/doc/harmonyos-guides/data-persistence-by-kv-store)

## 5. Preferences Data Persistence (Below Several KB) — pref

Suitable for storing lightweight configurations (such as user preferences and settings), implemented based on `@ohos.data.preferences`.
- Initialization: `pref.init(this.context)`
- Put/Get/Delete: `put(key, value)`, `get(key, defVal)`, `delete(key)`
- Related API: [Preferences Storage](https://developer.huawei.com/consumer/en/doc/harmonyos-guides/data-persistence-by-preferences)

## 6. Best Practices and Official Documentation

- Utility classes should follow the single responsibility principle and have simple interfaces for easy reuse and maintenance.
- For important capabilities such as device ID and data storage, always refer to the [Official Documentation](https://developer.huawei.com/consumer/en/doc/harmonyos-guides/arkts-coding-style-guide) and [ArkTS Coding Style Guide](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-coding-style-guide-V5).
- Code should include detailed comments, and key exceptions should be logged for troubleshooting.

**All content for this chapter is complete! The full source code has been uploaded to Gitee: [HarmonyOS App 0-1 Development](https://gitee.com/qincji/ZeroOneApp).** 