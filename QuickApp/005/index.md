# Part 5: Implementing Immersive Effects in HarmonyOS

In HarmonyOS app development, immersive experience is an important way to enhance UI aesthetics and user interaction. Immersive effects usually mean that page content can extend into the system status bar and navigation bar areas, creating a borderless visual experience. This article details how to implement immersive effects in HarmonyOS ArkTS projects, including full-screen settings, status/navigation bar adaptation, and reusable gradient header components.

## 1. Key Steps to Achieve Immersive Effects

### 1. Set Full-Screen Window and System Bar Properties

HarmonyOS provides various window management APIs. For immersive effects, it is recommended to use a "full-screen layout + transparent/custom system bar" approach:

```ts
let isLayoutFullScreen = true;
windowClass.setWindowLayoutFullScreen(isLayoutFullScreen, (err: BusinessError) => {
  if (err.code) {
    console.error('Failed to set the window layout to full-screen mode. Cause:' + JSON.stringify(err));
    return;
  }
  console.info('Succeeded in setting the window layout to full-screen mode.');
});
let sysBarProps: window.SystemBarProperties = {
  statusBarColor: '#ff00ff',
  navigationBarColor: '#00ff00',
  statusBarContentColor: '#ffffff',
  navigationBarContentColor: '#ffffff'
};
windowClass.setWindowSystemBarProperties(sysBarProps, (err: BusinessError) => {
  if (err.code) {
    console.error('Failed to set the system bar properties. Cause: ' + JSON.stringify(err));
    return;
  }
  console.info('Succeeded in setting the system bar properties.');
});
```

### 2. Obtain Status Bar and Navigation Bar Heights

In the `aboutToAppear` lifecycle of the `EntryView` page, use `window.getLastWindow(getContext())` to get the window object, then use `getWindowAvoidArea` to get the heights of the status bar and navigation bar, and store them in global variables:

```ts
aboutToAppear(): void {
  window.getLastWindow(getContext()).then((lastWindow) => {
    let areas = lastWindow.getWindowAvoidArea(window.AvoidAreaType.TYPE_SYSTEM);
    const statusHeight = px2vp(areas.topRect.height);
    areas = lastWindow.getWindowAvoidArea(window.AvoidAreaType.TYPE_NAVIGATION_INDICATOR);
    const navHeight = px2vp(areas.bottomRect.height);
    AppStorage.setOrCreate('statusHeight', statusHeight);
    AppStorage.setOrCreate('navHeight', navHeight);
  });
}
```

### 3. Page Adaptation and Placeholder Handling

In pages that need adaptation (such as `MainPage`, `HomePage`), use the `@StorageProp` decorator to get the global height variables, and reserve space for the status bar and navigation bar in the layout:

```ts
@StorageProp('statusHeight') statusHeight: number = 0;
@StorageProp('navHeight') navHeight: number = 0;
...
Line().height(this.navHeight).visibility(Visibility.Hidden)
```

### 4. Gradient Immersive Header Component

By customizing a `Header` component with linear gradient, status bar height placeholder, back button, and right-side custom layout, you can achieve a highly reusable immersive header:

```ts
@Component
export struct Header {
  @Require @Prop title: string | Resource;
  onKeyBack?: () => void;
  @BuilderParam rightLayout?: () => void;
  titleBarHeight: Length = 45;
  titleSize: number | string | Resource = '18fp';
  titleAttrModifier: AttributeModifier<TextAttribute> = {};
  bgTopColor: ResourceColor = '#74C678';
  bgBottomColor: ResourceColor = '#266B29';
  titleColor: ResourceColor = Color.White;
  @StorageProp('statusHeight') statusHeight: number = 0;
  build() {
    Stack() {
      RelativeContainer() {
        Text(this.title)
          .fontSize(this.titleSize)
          .width('50%')
          .height('100%')
          .fontColor(this.titleColor)
          .fontWeight(FontWeight.Medium)
          .ellipsisMode(EllipsisMode.END)
          .textOverflow({ overflow: TextOverflow.Ellipsis })
          .maxLines(1)
          .textAlign(TextAlign.Center)
          .alignRules({ middle: { anchor: "__container__", align: HorizontalAlign.Center } })
          .id("i1")
          .attributeModifier(this.titleAttrModifier)
        if (this.onKeyBack) {
          Image($r('app.media.ic_back'))
            .height('100%')
            .padding({ left: 16, top: 11, bottom: 11, right: 11 })
            .fillColor(this.titleColor)
            .objectFit(ImageFit.Contain)
            .alignRules({ left: { anchor: "__container__", align: HorizontalAlign.Start } })
            .id("i2")
            .onClick(() => { this.onKeyBack?.() })
        }
        if (this.rightLayout) {
          Row() { this.rightLayout?.() }
            .height('100%')
            .justifyContent(FlexAlign.End)
            .alignRules({ right: { anchor: "__container__", align: HorizontalAlign.End } })
            .id("i3")
        }
      }
      .width('100%')
      .height(this.titleBarHeight)
    }
    .width('100%')
    .padding({ top: this.statusHeight })
    .backgroundColor(this.bgTopColor)
    .linearGradient({
      angle: 0,
      colors: [ [this.bgTopColor, 0.0], [this.bgBottomColor, 1.0] ]
    })
  }
}
```

## 2. Best Practices and Official Documentation

- Recommended reading: [Window Management and Immersive Effects Official Guide](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-develop-apply-immersive-effects-V5)
- [ArkTS Coding Style Guide](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-coding-style-guide-V5)
- It is recommended to encapsulate the immersive header as a common component for reuse across multiple pages.
- Store status bar and navigation bar heights globally to avoid redundant calculations.
- Visual details such as gradients, rounded corners, and transparency can be flexibly adjusted according to product style.

> Note: The state management in this article uses V1. The open-source project has been upgraded to V2. Please upgrade accordingly.

**All content for this chapter is complete! The full source code has been uploaded to Gitee: [HarmonyOS App 0-1 Development](https://gitee.com/qincji/ZeroOneApp).** 