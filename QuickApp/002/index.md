# Part 2: How to Build Routing Pages with Navigation and Tab in HarmonyOS?

In HarmonyOS app development, page routing and multi-page management are essential capabilities. HarmonyOS NEXT recommends using the [Navigation](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-navigation-navigation-V5) component for page navigation and stack management, combined with Tab for multi-page switching. This article provides a detailed guide on how to use Navigation and Tab components to build a complete routing structure: splash page → main page (multi-Tab) → secondary pages.

## 1. Define a Global Unique @Entry

After creating a new ArkTS project, it is recommended to rename the default `Index` page to `EntryView`, and update related configurations in `ets/entryability/EntryAbility`, `ets/resources/base/profile/main_pages`, etc. Core code example:

```ts
@Entry
@Component
struct EntryView {
    build() {
        // Page content
    }
}
```

## 2. Implementing a Global Unique Navigation

HarmonyOS recommends maintaining only one global Navigation instance to manage the page stack. Implementation steps:

### 1. Create a Route Management Object

```ts
@Provide('pageStack') pageStack: NavPathStack = new NavPathStack();
```

Other pages can use `@Consume('pageStack')` to access this object and perform page navigation:

```ts
@Consume('pageStack') pageStack: NavPathStack
const navPath: NavPathInfo = new NavPathInfo('AboutMePage', undefined);
this.pageStack.pushPath(navPath);
```

### 2. Define a Page Mapping Builder

Use the @Builder decorator to dynamically return the corresponding page component based on the name parameter:

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

### 3. Implement Navigation in build()

```ts
build() {
    Stack() {
        Navigation(this.pageStack) {
            // Main page component
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

## 3. Implementing Tab + Main Page

### 1. Create Three Tab Page Components

For example, the Home page:

```ts
@Component
struct HomePage {
    build() {
        Column() {
            Text('This is the Home Page').fontSize(35)
        }
        .width('100%')
        .height('100%')
    }
}
```

### 2. Create a Custom Tab Bar

```ts
@Builder
tabBuilder() {
    Row() {
        ForEach(['Home', 'Tools', 'Mine'], (item: string, index) => {
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

### 3. Combine Navigation and Tab

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

> Note: The above approach means the previous page will be destroyed when switching tabs. If you want to retain page state, you can use a MainPage to manage all Tab pages.

## 4. Best Practices and Official Documentation

- Recommended reading: [Navigation Official Documentation](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-navigation-navigation-V5)
- [ArkUI Component Development Guide](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-ui-development-V5)
- [ArkTS Coding Style Guide](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-coding-style-guide-V5)
- It is recommended to use NavigationMode.Stack for stack management (other modes may cause split view effects, which are not beginner-friendly).
- The Tab bar can be customized for better user experience.

The next article will introduce how to further encapsulate the Navigation routing framework for more flexible page management.

**All content for this chapter is complete! The full source code has been uploaded to Gitee: [HarmonyOS App 0-1 Development](https://gitee.com/qincji/ZeroOneApp).** 