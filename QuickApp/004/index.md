# Part 4: How to Implement a Dialog That Blocks the System Back Key in HarmonyOS?

In HarmonyOS app development, dialogs are a very common interaction pattern, especially in scenarios where users must confirm actions or view system announcements. Sometimes, developers want to prevent dialogs from being closed by the system back key. The ArkTS `@CustomDialog` component does not support intercepting the system back key, but you can achieve this by using the [Navigation](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-navigation-navigation-V5) component's `NavDestination` to intercept the back event and create dialogs that cannot be dismissed arbitrarily.

## 1. Core Implementation Approach

1. Use the `NavDestination` component and set `.mode(NavDestinationMode.DIALOG)` to display the page as a dialog.
2. Intercept the system back event by returning `true` in `.onBackPressed(() => true)`, which prevents the dialog from being closed by the back key.
3. Set `.backgroundColor("#99000000")` to create a semi-transparent overlay effect.

Example core code:

```ts
NavDestination() {
  Stack() {
    // Custom dialog content
  }
  .backgroundColor("#99000000")
  .width('100%')
  .height('100%')
}
.onBackPressed(() => {
  return true; // Intercept back
})
.mode(NavDestinationMode.DIALOG)
.hideTitleBar(true)
```

## 2. Implementing a Reusable Project-Level Dialog

### 1. Create a dialog module and configure routing
- Create a `dialog` module under the `common` directory, and configure dependencies and dynamic import.
- Register the route in `RouterPage`:

```ts
static readonly HINT_DIALOG: RouterInfo = RouterPage.create("@common/dialog", "hint_dialog", true);
```

- Implement `harInit` in the `Index.ets` file of the `dialog` module to dynamically import the page based on `pageName`:

```ts
export function harInit(pageName: string) {
  switch (pageName) {
    case RouterPage.HINT_DIALOG.pageName:
      import('./src/main/ets/components/hint/HintDialog');
      break;
  }
}
```

- Register the page in the `HintDialog` file:

```ts
RouterNav.registerPage(RouterPage.HINT_DIALOG, wrapBuilder(getPage))
```

### 2. Component Implementation and Usage

```ts
@Component
export struct HintDialog {
  @State result: string = "";
  build() {
    NavDestination() {
      Stack() {
        Button("This is a dialog that intercepts the system back key. Only clicking me (pop) can close the dialog.")
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

Usage example:

```ts
Button('Dialog')
  .onClick(() => {
    RouterNav.push(RouterPage.HINT_DIALOG)
  })
```

## 3. Custom Dialog Parameters and Global Styles

- By passing a `HintParm` parameter object, you can flexibly customize dialog content, buttons, callbacks, styles, and more:

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

- Support for global custom styles via `HintDialog.initDefaultStyle(HintStyle)`:

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

- Example:

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
  titleParm: new TitleParm('Default Hint', titleAttr),
  outBoxAttr: outBoxAttr
})
```

## 4. Best Practices and Official Documentation

- Recommended reading: [Navigation Official Documentation](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-navigation-navigation-V5)
- [ArkTS Coding Style Guide](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-coding-style-guide-V5)
- It is recommended to encapsulate dialogs as reusable components with flexible parameters and styles.
- When intercepting the back key, always provide a clear way to close the dialog (such as an "OK" button) to avoid trapping the user.
- Global style configuration can improve product consistency and development efficiency.

**All content for this chapter is complete! The full source code has been uploaded to Gitee: [HarmonyOS App 0-1 Development](https://gitee.com/qincji/ZeroOneApp).** 