# 第6篇：Navigation系统路由表（推荐）

在鸿蒙应用开发中，页面路由管理是多页面项目的核心。自定义路由模块虽然灵活，但配置复杂、维护成本高。自API 12起，HarmonyOS官方提供了[系统路由表](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-navigation-navigation-V5#系统路由表)能力，极大简化了页面注册与跳转流程。本文将详细介绍如何使用系统路由表实现高效、规范的页面跳转。

## 一、系统路由表的基本用法

### 1. 在目标模块配置module.json5

无论是`entry`还是业务模块（如`login`），都可在`module.json5`中添加如下配置：

```json
{
  "module": {
    "routerMap": "$profile:route_map"
  }
}
```

### 2. 新建route_map.json并注册页面

在`resources/base/profile`目录下新建`route_map.json`，以`login`模块为例：

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

- `name`：页面名称，后续跳转时使用。
- `pageSourceFile`：页面源码路径，相对`src`目录。
- `buildFunction`：页面入口函数，需用`@Builder`修饰。
- `data`：自定义字段，可通过`getConfigInRouteMap`接口获取。

### 3. 页面入口函数注册

在目标页面（如`LoginPage.ets`）中，添加`@Builder`修饰的注册函数：

```ts
// 跳转页面入口函数
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

### 4. 通过pushPathByName接口跳转页面

在页面中通过`pushPathByName`即可实现页面跳转，无需手动维护navDestination：

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

## 二、项目结构优化与路由工具类

### 1. 版本要求

- 需API 12（beta1）及以上，`build-profile.json5`配置：

```json
"compileSdkVersion": "5.0.0(12)",
"compatibleSdkVersion": "5.0.0(12)",
```

### 2. 移除自定义router模块

- 系统路由表已满足大部分需求，可移除自定义router模块及相关引用，简化项目结构。

### 3. 推荐封装RouterUtil工具类

为便于统一管理页面跳转、返回、替换等操作，建议封装`RouterUtil`工具类：

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

## 三、最佳实践与官方文档

- 推荐阅读[Navigation官方文档-系统路由表章节](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-navigation-navigation-V5#系统路由表)和[ArkTS编程规范](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-coding-style-guide-V5)。
- 路由表建议统一维护，页面注册与跳转规范化，便于多人协作。
- 工具类可根据实际业务扩展，提升开发效率。

**到这里，所有的内容已经结束了！本章的完整源码已经上传到gitee了：[鸿蒙应用0-1开发](https://gitee.com/qincji/ZeroOneApp)。**