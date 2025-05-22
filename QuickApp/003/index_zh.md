# 第3篇：手把手教你如何实现对Navigation路由框架的封装！

在前两篇文章中，我们已经掌握了[Navigation](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-navigation-navigation-V5)组件的基本用法和Tab多页面管理。本篇将进一步带你实现对Navigation路由框架的封装，使其适配更复杂的项目结构，支持动态模块加载、页面参数传递、回调等高级能力，提升项目的可维护性和扩展性。

## 一、路由封装的基本思路

在实际开发中，随着项目规模扩大，页面数量和模块会不断增加。为了便于管理和解耦，推荐将路由管理逻辑进行统一封装。核心思路如下：
- 通过`RouterInfo`类描述每个页面的模块名、页面名、是否需要`NavDestination`等信息。
- 通过`RouterPage`统一管理所有页面信息，便于查找和维护。
- 通过`RouterNav`实现页面注册、动态导入、页面跳转、参数传递、回调等功能。
- 支持[动态import](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-dynamic-import-V5)按需加载模块，提升性能。

## 二、核心代码结构与实现

### 1. 路由页面信息描述

```ts
export interface RouterInfo {
  moduleName: string //模块名称
  pageName: string //页面名称
  hasNavDest?: boolean //是否包含NavDestination
}
```

### 2. 页面统一管理类

```ts
export class RouterPage {
  private static infos: Map<string, RouterInfo> = new Map<string, RouterInfo>();
  static readonly LOGIN: RouterInfo = RouterPage.create("@module/login", "login", true);
  static readonly REGISTER: RouterInfo = RouterPage.create("@module/login", "register");
  static readonly ENTRY_MODULE_NAME: string = "@module/entry";
  static readonly ABOUT_ME: RouterInfo = RouterPage.create("@module/entry", "about_me", true);
  static readonly ABOUT_ME2: RouterInfo = RouterPage.create("@module/entry", "about_me2");
  // ... 省略其他页面
  private static create(moduleName: string, pageName: string, hasNavDest?: boolean): RouterInfo {
    const path = moduleName + "/" + pageName;
    let info = RouterPage.infos.get(path);
    if (info) return info;
    info = { moduleName, pageName, hasNavDest };
    RouterPage.infos.set(path, info);
    return info;
  }
  public static getInfo(path: string) { return RouterPage.infos.get(path); }
  public static getPath(info: RouterInfo) { return info.moduleName + "/" + info.pageName; }
}
```

### 3. 路由管理与动态注册

```ts
export class RouterNav {
  private static routerMap: Map<string, WrappedBuilder<[object]>> = new Map();
  static navPathStack: NavPathStack = new NavPathStack();
  private static routerStack: Array<RouterInfo> = [];
  public static registerPage(info: RouterInfo, builder: WrappedBuilder<[object]>) {
    RouterNav.routerMap.set(RouterPage.getPath(info), builder);
  }
  public static getBuilder(name: string): WrappedBuilder<[object]> {
    return RouterNav.routerMap.get(name) as WrappedBuilder<[object]>;
  }
  public static hasNavDest(name: string): boolean {
    const info = RouterPage.getInfo(name);
    return info?.hasNavDest ?? false;
  }
  public static async push(info: RouterInfo, param: object | undefined = undefined,
    navPath: NavPathInfo = new NavPathInfo(RouterPage.getPath(info), param)) {
    const moduleName = info.moduleName;
    const pageName = info.pageName;
    let isImportSucceed = false;
    if (moduleName === RouterPage.ENTRY_MODULE_NAME) {
      isImportSucceed = true;
    } else {
      await import(moduleName).then((result: ESObject) => {
        result.harInit(pageName);
        isImportSucceed = true;
      }, (error: ESObject) => {
        // 错误处理
      });
    }
    if (isImportSucceed) {
      RouterNav.routerStack.push(info);
      RouterNav.navPathStack.pushPath(navPath);
    }
  }
  public static pop(result?: Object) {
    if (result !== undefined) {
      RouterNav.navPathStack.pop(result, true)
    } else {
      RouterNav.navPathStack.pop(true)
    }
  }
}
```

### 4. 动态import与页面注册

- 在每个模块的Index.ets中实现`harInit`，根据pageName动态导入页面：

```ts
export function harInit(pageName: string) {
  switch (pageName) {
    case RouterPage.LOGIN.pageName:
      import('./src/main/ets/pages/LoginPage');
      break;
    // ... 其他页面
  }
}
```
- 在页面文件中注册页面：

```ts
RouterNav.registerPage(RouterPage.LOGIN, wrapBuilder(getPage))
```

### 5. 页面跳转与参数传递

- 跳转页面并传递参数：

```ts
Button('登录').onClick(() => {
  const record = { from: 'entry/home', text: 'hello', age: 18 };
  RouterNav.push(RouterPage.LOGIN, record)
})
```
- 在目标页面获取参数：

```ts
@Component
export struct LoginPage {
  @State result: string = "";
  build() {
    NavDestination() {
      Column() {
        Text('这是登录页面, 有NavDestination').fontSize(35)
        Text(`接收到的数据：${this.result}`).fontSize(35)
      }
    }
    .onReady(cxt => {
      const record = cxt.pathInfo.param as Record<string, object>;
      this.result = JSON.stringify(record);
    })
  }
}
```
- 支持回调返回数据：

```ts
Button('登录--》传些参数看看').onClick(() => {
  const record = { from: 'entry/home', text: 'hello', age: 18 };
  RouterNav.pushCallback({
    name: RouterPage.getPath(RouterPage.LOGIN),
    param: record,
    onPop: async (info: PopInfo) => {
      // 取得回来的参数
      let params = (info.result as Record<string, Object>);
    }
  })
})
```

## 三、最佳实践与官方文档

- 推荐阅读[Navigation官方文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-navigation-navigation-V5)
- [动态import官方文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-dynamic-import-V5)
- [ArkTS编程规范](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-coding-style-guide-V5)。
- 路由管理类建议放在`common`目录，页面模块建议按功能拆分，提升可维护性。
- 动态import可有效减少主包体积和内存占用，适合大型项目。
- 参数传递和回调机制可极大提升页面间交互灵活性。

**到这里，所有的内容已经结束了！本章的完整源码已经上传到gitee了：[鸿蒙应用0-1开发](https://gitee.com/qincji/ZeroOneApp)。**