# 第4篇：对话框是每个项目的基础，那么禁止系统返回的对话框应该如何实现呢？

在鸿蒙应用开发中，弹窗（Dialog）是非常常见的交互方式，尤其在需要用户强制确认、系统公告等场景下，开发者常常希望弹窗无法被系统返回键关闭。HarmonyOS ArkTS 的 `@CustomDialog` 组件本身不支持拦截系统返回键，但通过[Navigation](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-navigation-navigation-V5)的 `NavDestination` 组件，可以实现对系统返回的拦截，打造不可随意关闭的对话框。

## 一、核心实现思路

1. 使用 `NavDestination` 组件，并设置 `.mode(NavDestinationMode.DIALOG)`，让页面以对话框模式弹出。
2. 通过 `.onBackPressed(() => true)` 拦截系统返回事件，返回 `true` 即可禁止返回键关闭弹窗。
3. 主动设置 `.backgroundColor("#99000000")`，实现半透明遮罩效果。

核心代码示例：

```ts
NavDestination() {
  Stack() {
    // 自定义弹窗内容
  }
  .backgroundColor("#99000000")
  .width('100%')
  .height('100%')
}
.onBackPressed(() => {
  return true; // 拦截返回
})
.mode(NavDestinationMode.DIALOG)
.hideTitleBar(true)
```

## 二、项目级可复用对话框的实现

### 1. 新建 dialog 模块与路由配置
- 在 `common` 目录下新建 `dialog` 模块，并配置依赖和动态 import。
- 在 `RouterPage` 中注册路由信息：

```ts
static readonly HINT_DIALOG: RouterInfo = RouterPage.create("@common/dialog", "hint_dialog", true);
```

- 在 `dialog` 模块的 `Index.ets` 文件中实现 `harInit`，根据 pageName 动态导入页面：

```ts
export function harInit(pageName: string) {
  switch (pageName) {
    case RouterPage.HINT_DIALOG.pageName:
      import('./src/main/ets/components/hint/HintDialog');
      break;
  }
}
```

- 在 `HintDialog` 页面文件中注册页面：

```ts
RouterNav.registerPage(RouterPage.HINT_DIALOG, wrapBuilder(getPage))
```

### 2. 组件实现与调用

```ts
@Component
export struct HintDialog {
  @State result: string = "";
  build() {
    NavDestination() {
      Stack() {
        Button("这是能拦截系统返回的弹窗，只有点我调用pop才能关闭弹窗")
          .onClick(() => {
            RouterNav.pop();
          })
      }
      .backgroundColor("#99000000")
      .width('100%')
      .height('100%')
    }
    .onReady(cxt => {
      const record = cxt.pathInfo.param as Record<string, object>;
      this.result = JSON.stringify(record);
    })
    .onBackPressed(() => true)
    .mode(NavDestinationMode.DIALOG)
    .hideTitleBar(true)
  }
}
```

调用方式：

```ts
Button('对话框')
  .onClick(() => {
    RouterNav.push(RouterPage.HINT_DIALOG)
  })
```

## 三、自定义弹窗参数与全局样式

- 通过传递 `HintParm` 参数对象，可灵活定制弹窗内容、按钮、回调、样式等：

```ts
export interface HintParm {
  content: string
  title?: string
  noTitle?: boolean
  noMsg?: string
  okMsg?: string
  showNo?: boolean
  isWebType?: boolean
  controller?: WebviewController
  pressBackCancel?: boolean
  outsideCancel?: boolean
  onNo?: () => void
  onOk?: () => void
  alignment?: Alignment
}
```

- 支持全局自定义样式，通过 `HintDialog.initDefaultStyle(HintStyle)` 设置：

```ts
export interface HintStyle {
  outsideCancel?: boolean
  pressBackCancel?: boolean
  alignment?: Alignment
  titleParm?: TitleParm
  textParm?: TextParm
  webParm?: WebParm
  btnParm?: BtnParm
  outBoxAttr?: AttributeModifier<ColumnAttribute>
  lineStyleTop?: AttributeModifier<LineAttribute>
  lineStyleBottom?: AttributeModifier<LineAttribute>
}
```

- 示例：

```ts
const titleAttr: AttributeModifier<TextAttribute> = {
  applyNormalAttribute(instance: TextAttribute): void {
    instance.fontColor('#ff1145e5').fontWeight(FontWeight.Bolder).fontSize(25)
  }
};
const outBoxAttr: AttributeModifier<ColumnAttribute> = {
  applyNormalAttribute(instance: ColumnAttribute): void {
    instance.backgroundColor('#ffc8f1b3').borderRadius(5).margin(50)
  }
};
HintDialog.initDefaultStyle({
  titleParm: new TitleParm('默认提示', titleAttr),
  outBoxAttr: outBoxAttr
})
```

## 四、最佳实践与官方文档

- 推荐阅读[Navigation官方文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-navigation-navigation-V5)
- [ArkTS编程规范](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-coding-style-guide-V5)
- 对话框建议统一封装为可复用组件，参数和样式灵活可配。
- 拦截返回键时，务必提供明确的关闭方式（如"确定"按钮），避免用户无法退出。
- 全局样式配置可提升产品一致性和开发效率。

**到这里，所有的内容已经结束了！本章的完整源码已经上传到gitee了：[鸿蒙应用0-1开发](https://gitee.com/qincji/ZeroOneApp)。**