# Part 3: Step-by-Step Guide to Encapsulating the Navigation Routing Framework in HarmonyOS

In the previous articles, we have mastered the basic usage of the [Navigation](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-navigation-navigation-V5) component and multi-page management with Tabs. This article will further guide you to encapsulate the Navigation routing framework, making it suitable for more complex project structures, supporting dynamic module loading, page parameter passing, callbacks, and other advanced features to improve maintainability and scalability.

## 1. Core Ideas for Routing Encapsulation

As your project grows, the number of pages and modules will increase. To facilitate management and decoupling, it is recommended to encapsulate routing logic in a unified way. The core ideas are as follows:
- Use a `RouterInfo` class to describe each page's module name, page name, and whether it requires `NavDestination`.
- Use `RouterPage` to centrally manage all page information for easy lookup and maintenance.
- Use `RouterNav` to implement page registration, dynamic import, navigation, parameter passing, callbacks, and more.
- Support [dynamic import](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-dynamic-import-V5) for on-demand module loading and better performance.

## 2. Key Code Structure and Implementation

### 1. Page Information Description

```ts
export interface RouterInfo {
  moduleName: string; // Module name
  pageName: string;   // Page name
  hasNavDest?: boolean; // Whether it contains NavDestination
}
```

### 2. Centralized Page Management Class

```ts
export class RouterPage {
  private static infos: Map<string, RouterInfo> = new Map<string, RouterInfo>();
  static readonly LOGIN: RouterInfo = RouterPage.create("@module/login", "login", true);
  static readonly REGISTER: RouterInfo = RouterPage.create("@module/login", "register");
  static readonly ENTRY_MODULE_NAME: string = "@module/entry";
  static readonly ABOUT_ME: RouterInfo = RouterPage.create("@module/entry", "about_me", true);
  static readonly ABOUT_ME2: RouterInfo = RouterPage.create("@module/entry", "about_me2");
  // ... other pages omitted
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

### 3. Routing Management and Dynamic Registration

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
        // Error handling
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

### 4. Dynamic Import and Page Registration

- In each module's Index.ets, implement `harInit` to dynamically import pages based on pageName:

```ts
export function harInit(pageName: string) {
  switch (pageName) {
    case RouterPage.LOGIN.pageName:
      import('./src/main/ets/pages/LoginPage');
      break;
    // ... other pages
  }
}
```
- Register the page in the page file:

```ts
RouterNav.registerPage(RouterPage.LOGIN, wrapBuilder(getPage))
```

### 5. Page Navigation and Parameter Passing

- Navigate to a page and pass parameters:

```ts
Button('Login').onClick(() => {
  const record = { from: 'entry/home', text: 'hello', age: 18 };
  RouterNav.push(RouterPage.LOGIN, record)
})
```
- Retrieve parameters in the target page:

```ts
@Component
export struct LoginPage {
  @State result: string = "";
  build() {
    NavDestination() {
      Column() {
        Text('This is the Login Page with NavDestination').fontSize(35)
        Text(`Received data: ${this.result}`).fontSize(35)
      }
    }
    .onReady(cxt => {
      const record = cxt.pathInfo.param as Record<string, object>;
      this.result = JSON.stringify(record);
    })
  }
}
```
- Support for callback and returning data:

```ts
Button('Login → Pass Some Params').onClick(() => {
  const record = { from: 'entry/home', text: 'hello', age: 18 };
  RouterNav.pushCallback({
    name: RouterPage.getPath(RouterPage.LOGIN),
    param: record,
    onPop: async (info: PopInfo) => {
      // Get returned parameters
      let params = (info.result as Record<string, Object>);
    }
  })
})
```

## 3. Best Practices and Official Documentation

- Recommended reading: [Navigation Official Documentation](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-navigation-navigation-V5)
- [Dynamic Import Official Documentation](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-dynamic-import-V5)
- [ArkTS Coding Style Guide](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-coding-style-guide-V5)
- It is recommended to place routing management classes in the `common` directory and split page modules by feature for better maintainability.
- Dynamic import can effectively reduce the main package size and memory usage, suitable for large projects.
- Parameter passing and callback mechanisms greatly enhance the flexibility of page interactions.

**All content for this chapter is complete! The full source code has been uploaded to Gitee: [HarmonyOS App 0-1 Development](https://gitee.com/qincji/ZeroOneApp).** 