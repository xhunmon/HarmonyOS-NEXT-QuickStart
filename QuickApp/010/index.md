# Part 10: Migrating State Management from V1 to V2

> V2 is a must-have!

In HarmonyOS app development, state management is the core capability for synchronizing data between pages and sharing global variables. Since ArkTS V2, the official state management mechanism has been comprehensively upgraded, providing a more efficient and user-friendly global state management solution. This article combines the [official migration guide](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-v1-v2-migration-V5) and practical project experience to systematically explain the migration from V1 to V2, unified global variable management, global state listening and notification, and best practices.

## 1. Official Migration Advice and Project Comparison

- Official detailed migration guide: [ArkTS State Management V1-V2 Migration Guide](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-v1-v2-migration-V5)
- It is recommended to compare the project structure before and after migration, using git branches or new projects for comparison.
- V2 supports stronger global state synchronization, type safety, and automatic notification. All new and maintained projects are advised to migrate as soon as possible.

## 2. New Approach to Unified Global Variable Management

V2 recommends using `AppStorageV2` + `@ObservedV2` + `@Trace` for unified global variable management, greatly simplifying state synchronization across multiple pages/components.

```ts
import { AppStorageV2 } from '@kit.ArkUI';

@ObservedV2
export class V2Info {
  @Trace isLogin: boolean = false; // Login status
  @Trace statusHeight: number = 0; // Status bar height
  @Trace bottomHeight: number = 0; // Navigation bar height
  @Trace screenHeight: number = 0; // Screen height
  @Trace screenWidth: number = 0; // Screen width

  static connect(): V2Info {
    return AppStorageV2.connect<V2Info>(V2Info, () => new V2Info())!;
  }
}

// Listen in page/component
@ComponentV2
struct HomePage {
    @Local info: V2Info = V2Info.connect();
    ...
}

// Assign globally and auto-notify
V2Info.connect().isLogin = true;
```

## 3. Global State Listening and Notification Utility Class

By using a global object cache utility class (such as Global), you can listen to and notify any state, suitable for scenarios like splash screens and login.

```ts
// Listen
@Local splashFinish: boolean = global.statusChange('splashFinish', false, (value) => {
  this.splashFinish = value
  global.removeObject('splashFinish');
})

// Notify
global.statusNotify('splashFinish', true);
```

## 4. Best Practices and Official Documentation

- Recommended reading: [ArkTS State Management V2 Official Documentation](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-state-management-V5) and [ArkTS Coding Style Guide](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-coding-style-guide-V5).
- It is recommended to manage global state centrally to avoid duplicate definitions.
- Listening and notification should be encapsulated as utility classes for reuse and maintenance.
- Migrate in stages and by module to ensure a smooth transition.
- Code should include detailed comments, and key exceptions should be logged for troubleshooting.

**The complete code is hosted at [HarmonyOS App 0-1 Development](https://gitee.com/qincji/ZeroOneApp). Welcome to Star & Fork!** 