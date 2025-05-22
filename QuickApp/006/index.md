# Part 6: Navigation System Route Table (Recommended)

In HarmonyOS app development, page routing management is the core of multi-page projects. While custom routing modules are flexible, they are often complex to configure and maintain. Since API 12, HarmonyOS officially provides a [system route table](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-navigation-navigation-V5#系统路由表) feature, which greatly simplifies page registration and navigation. This article details how to use the system route table for efficient and standardized page navigation.

## 1. Basic Usage of the System Route Table

### 1. Configure `module.json5` in the Target Module

Whether in the `entry` or a business module (such as `login`), add the following configuration to `module.json5`:

```json
{
  "module": {
    "routerMap": "$profile:route_map"
  }
}
```

### 2. Create `route_map.json` and Register Pages

Create `route_map.json` in the `resources/base/profile` directory. For example, in the `login` module:

```json
{
  "routerMap": [
    {
      "name": "LoginPage",
      "pageSourceFile": "src/main/ets/pages/LoginPage.ets",
      "buildFunction": "RegisterBuilder",
      "data": {
        "description": "this is LoginPage"
      }
    }
  ]
}
```

- `name`: The page name, used for navigation.
- `pageSourceFile`: The source file path of the page, relative to the `src` directory.
- `buildFunction`: The entry function for the page, must be decorated with `@Builder`.
- `data`: Custom fields, accessible via the `getConfigInRouteMap` API.

### 3. Register the Page Entry Function

In the target page (e.g., `LoginPage.ets`), add a function decorated with `@Builder`:

```ts
// Page entry function for navigation
@Builder
export function RegisterBuilder() {
  LoginPage()
}

@Component
struct LoginPage {
  pathStack: NavPathStack = new NavPathStack()
  build() {
    NavDestination() {}
    .onReady((context: NavDestinationContext) => {
      this.pathStack = context.pathStack
    })
  }
}
```

### 4. Navigate Between Pages Using `pushPathByName`

You can navigate between pages using `pushPathByName`, without manually maintaining `navDestination`:

```ts
pageStack: NavPathStack = new NavPathStack();
build() {
  Navigation(this.pageStack){}
    .onAppear(() => {
      this.pageStack.pushPathByName("LoginPage", null, false);
    })
    .hideNavBar(true)
}
```

## 2. Project Structure Optimization and Routing Utility Class

### 1. Version Requirements

- Requires API 12 (beta1) or above. In `build-profile.json5`:

```json
"compileSdkVersion": "5.0.0(12)",
"compatibleSdkVersion": "5.0.0(12)",
```

### 2. Remove Custom Router Module

- The system route table covers most needs, so you can remove the custom router module and related references to simplify the project structure.

### 3. Recommended: Encapsulate a `RouterUtil` Utility Class

To manage navigation, back, and replace operations in a unified way, encapsulate a `RouterUtil` utility class:

```ts
export class RouterUtil {
  static navPathStack: NavPathStack = new NavPathStack();
  static routerStack: Array<string> = new Array();

  public static push(name: string, parm: object | undefined = undefined,
    callback: Callback<PopInfo> | undefined = undefined) {
    RouterUtil.routerStack.push(name);
    RouterUtil.navPathStack.pushPathByName(name, parm, callback, true)
  }

  public static replace(name: string, parm: object | undefined = undefined) {
    RouterUtil.routerStack.pop();
    RouterUtil.routerStack.push(name);
    RouterUtil.navPathStack.replacePathByName(name, parm, true)
  }

  public static pop(result?: Object) {
    if (result !== undefined) {
      RouterUtil.navPathStack.pop(result, true)
    } else {
      RouterUtil.navPathStack.pop(true)
    }
  }

  public static popHome(){
    RouterUtil.routerStack.length = 0;
    RouterUtil.navPathStack.clear()
  }
}
```

## 3. Best Practices and Official Documentation

- Recommended reading: [Navigation Official Documentation - System Route Table Section](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-navigation-navigation-V5#系统路由表)
- [ArkTS Coding Style Guide](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-coding-style-guide-V5)
- It is recommended to maintain the route table centrally, and standardize page registration and navigation for better team collaboration.
- The utility class can be extended as needed to improve development efficiency.

**All content for this chapter is complete! The full source code has been uploaded to Gitee: [HarmonyOS App 0-1 Development](https://gitee.com/qincji/ZeroOneApp).** 