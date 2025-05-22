# 第10篇：状态管理V1全面迁移至V2

> V2是必须的呀！

在鸿蒙应用开发中，状态管理是实现页面间数据同步、全局变量共享的核心能力。ArkTS自V2版本起，官方对状态管理机制进行了全面升级，带来了更高效、更易用的全局状态管理方案。本文将结合[官方迁移文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-v1-v2-migration-V5)和实际项目经验，系统讲解V1到V2的迁移思路、全局变量统一管理、全局状态监听与通知等最佳实践。

## 一、官方迁移建议与项目对比

- 官方详细迁移文档：[ArkTS状态管理V1-V2迁移指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-v1-v2-migration-V5)
- 建议对比迁移前后项目结构，推荐用git分支或新建工程对比差异。
- V2支持更强的全局状态同步、类型安全、自动通知等能力，建议所有新项目和维护项目尽快迁移。

## 二、全局变量统一管理新写法

V2推荐通过`AppStorageV2`+`@ObservedV2`+`@Trace`实现全局变量统一管理，极大简化了多页面/多组件间的状态同步。

```ts
import { AppStorageV2 } from '@kit.ArkUI';

@ObservedV2
export class V2Info {
  @Trace isLogin: boolean = false; //登录状态
  @Trace statusHeight: number = 0; //状态栏高度
  @Trace bottomHeight: number = 0; //导航栏高度
  @Trace screenHeight: number = 0; //屏幕高度
  @Trace screenWidth: number = 0; //屏幕宽度

  static connect(): V2Info {
    return AppStorageV2.connect<V2Info>(V2Info, () => new V2Info())!;
  }
}

// 页面/组件内监听
@ComponentV2
struct HomePage {
    @Local info: V2Info = V2Info.connect();
    ...
}

// 全局赋值并自动通知
V2Info.connect().isLogin = true;
```

## 三、全局状态监听与通知工具类

通过全局对象缓存工具类（如Global），可实现对任意状态的监听与单点通知，适合Splash、登录等场景。

```ts
// 监听
@Local splashFinish: boolean = global.statusChange('splashFinish', false, (value) => {
  this.splashFinish = value
  global.removeObject('splashFinish');
})

// 通知
global.statusNotify('splashFinish', true);
```

## 四、最佳实践与官方文档

- 推荐阅读[ArkTS状态管理V2官方文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-state-management-V5)和[ArkTS编程规范](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-coding-style-guide-V5)。
- 全局状态建议统一管理，避免多处重复定义。
- 监听与通知建议封装为工具类，便于复用和维护。
- 迁移时建议分阶段、分模块推进，确保功能平滑过渡。
- 代码应有详细注释，关键异常需日志记录，便于排查。

**完整代码已托管至[鸿蒙应用0-1开发](https://gitee.com/qincji/ZeroOneApp)。欢迎Star & Fork！**