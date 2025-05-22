# 第5篇：实现沉浸式效果

在鸿蒙应用开发中，沉浸式（Immersive）体验是提升用户界面美观性和交互感的重要手段。沉浸式效果通常指页面内容能够延伸到系统状态栏和导航栏区域，实现无边界的视觉体验。本篇将详细介绍如何在HarmonyOS ArkTS项目中实现沉浸式效果，包括全屏设置、状态栏/导航栏高度适配、渐变头部组件等实用技巧。

## 一、实现沉浸式的关键流程

### 1. 设置全屏窗口与系统栏属性

HarmonyOS提供了多种窗口管理API。实现沉浸式推荐采用"全屏布局+系统栏透明/自定义色"方案：

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

### 2. 获取状态栏和导航栏高度

在`EntryView`页面的`aboutToAppear`生命周期中，通过`window.getLastWindow(getContext())`获取窗口对象，再用`getWindowAvoidArea`分别获取状态栏和导航栏的高度，并存入全局变量：

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

### 3. 页面适配与占位处理

在需要适配的页面（如`MainPage`、`HomePage`），通过`@StorageProp`装饰器获取全局高度变量，并在布局中为状态栏和导航栏预留空间：

```ts
@StorageProp('statusHeight') statusHeight: number = 0;
@StorageProp('navHeight') navHeight: number = 0;
...
Line().height(this.navHeight).visibility(Visibility.Hidden)
```

### 4. 渐变色沉浸式头部组件

通过自定义`Header`组件，结合线性渐变、状态栏高度占位、返回按钮、右侧自定义布局等，实现高复用的沉浸式头部：

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

## 二、最佳实践与官方文档

- 推荐阅读[窗口管理与沉浸式开发官方文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-develop-apply-immersive-effects-V5)
- [ArkTS编程规范](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-coding-style-guide-V5)。
- 沉浸式头部建议封装为通用组件，便于多页面复用。
- 状态栏/导航栏高度建议全局存储，避免重复计算。
- 渐变色、圆角、透明度等视觉细节可根据产品风格灵活调整。

> 注意：本文使用的状态管理V1，开源项目已优化成V2，请对应升级。

**到这里，所有的内容已经结束了！本章的完整源码已经上传到gitee了：[鸿蒙应用0-1开发](https://gitee.com/qincji/ZeroOneApp)。**