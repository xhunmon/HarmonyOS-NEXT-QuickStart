# 第2篇：如何使用Navigation+Tab搭建路由页面？

在鸿蒙应用开发中，页面路由和多页面管理是非常核心的能力。HarmonyOS NEXT推荐使用[Navigation](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-navigation-navigation-V5)组件进行页面跳转和堆栈管理，并结合Tab实现多页面切换。本文将详细介绍如何通过Navigation和Tab组件，搭建一个启动页-主页（多Tab）-二级页面的完整路由结构。

## 一、定义全局唯一的@Entry

在新建ArkTS项目后，建议将默认的`Index`页面重命名为`EntryView`，并在`ets/entryability/EntryAbility`、`ets/resources/base/profile/main_pages`等相关配置中同步修改。核心代码如下：

```ts
@Entry
@Component
struct EntryView {
    build() {
        // 页面内容
    }
}
```

## 二、全局唯一Navigation的实现

HarmonyOS推荐全局只维护一个Navigation实例，统一管理页面堆栈。实现步骤如下：

### 1. 创建路由管理对象

```ts
@Provide('pageStack') pageStack: NavPathStack = new NavPathStack();
```

其他页面通过`@Consume('pageStack')`获取该对象，实现页面跳转：

```ts
@Consume('pageStack') pageStack: NavPathStack
const navPath: NavPathInfo = new NavPathInfo('AboutMePage', undefined);
this.pageStack.pushPath(navPath);
```

### 2. 定义页面映射Builder

通过@Builder装饰器，根据name参数动态返回对应页面组件：

```ts
@Builder
pageMap(name: string, param: object) {
    if (name === 'HomePage') {
        HomePage()
    } else if (name === 'ToolPage') {
        ToolPage()
    } else if (name === 'MyPage') {
        MyPage()
    }
}
```

### 3. 在build中实现Navigation

```ts
build() {
    Stack() {
        Navigation(this.pageStack) {
            // 主页组件
        }
        .height('100%')
        .hideTitleBar(true)
        .navBarWidth('50%')
        .navDestination(this.pageMap)
        .mode(NavigationMode.Stack)
    }
    .alignContent(Alignment.BottomEnd)
    .height('100%')
}
```

## 三、Tab+主页的实现

### 1. 创建三个Tab页面组件

以首页为例：

```ts
@Component
struct HomePage {
    build() {
        Column() {
            Text('这是首页').fontSize(35)
        }
        .width('100%')
        .height('100%')
    }
}
```

### 2. 创建自定义Tab栏

```ts
@Builder
tabBuilder() {
    Row() {
        ForEach(['首页', '工具', '我的'], (item: string, index) => {
            Column() {
                Text(item)
                    .fontColor(this.selectIndex === index ? Color.Red : Color.Black)
                    .fontSize(this.selectIndex === index ? 25 : 18)
            }
            .height('100%')
            .layoutWeight(1)
            .justifyContent(FlexAlign.End)
            .padding({ top: 5, bottom: 5 })
            .onClick(() => {
                this.selectIndex = index;
            })
        })
    }
    .height(50)
    .width('100%')
}
```

### 3. 组合Navigation与Tab

```ts
Navigation(this.pageStack) {
    if (this.selectIndex === 0) {
        HomePage()
    } else if (this.selectIndex === 1) {
        ToolPage()
    } else if (this.selectIndex === 2) {
        MyPage()
    }
}
.height('100%')
.hideTitleBar(true)
.navBarWidth('50%')
.navDestination(this.pageMap)
.mode(NavigationMode.Stack)
.hideNavBar(false)
.toolbarConfiguration(this.tabBuilder)
```

> 说明：上述方式适用于页面切换时前一个页面会被销毁。如果需要页面状态保留，可通过MainPage统一管理所有Tab页面。

## 四、最佳实践与官方文档

- 推荐阅读[Navigation官方文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-navigation-navigation-V5)

- [ArkUI组件开发指南](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-ui-development-V5)
- [ArkTS编程规范](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-coding-style-guide-V5)。
- 推荐使用NavigationMode.Stack（否者出现分栏效果，对初学者不好掌控）模式实现页面堆栈管理。
- Tab栏可自定义样式，提升用户体验。

下一篇将介绍如何进一步封装Navigation路由框架，实现更灵活的页面管理。

**到这里，所有的内容已经结束了！本章的完整源码已经上传到gitee了：[鸿蒙应用0-1开发](https://gitee.com/qincji/ZeroOneApp)。**