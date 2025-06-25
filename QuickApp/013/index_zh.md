# ZeroOneApp加载组件基础实现与使用

## 引言

在移动应用开发中，加载状态的展示是提升用户体验的关键环节。一个设计良好的加载组件能够有效缓解用户等待焦虑，提供清晰的交互反馈。ZeroOneApp的UI组件库中包含了一套功能完善的加载组件，基于HarmonyOS的ArkUI框架开发，支持多种加载样式、自定义配置和便捷的API调用。本文将从基础层面解析ZeroOneApp加载组件的实现原理，包括核心架构、样式定义和基本使用方法，帮助开发者快速掌握这一组件的集成与应用。

## 官方资源介绍

- **ArkUI组件开发指南**：HarmonyOS官方提供的UI组件开发规范和最佳实践[ArkUI组件开发文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkui)
- **Lottie动画库**：Airbnb开源的动画库，支持JSON格式的动画文件渲染[Lottie官方文档](https://lottiefiles.com/developers)
- **HarmonyOS Lottie适配库**：适用于HarmonyOS的Lottie动画实现[@ohos/lottie](https://ohpm.openharmony.cn/#/cn/detail/@ohos%2Flottie)

## 详细讲解

### 1. 加载组件核心架构

ZeroOneApp的加载组件采用模块化设计，主要由三个核心文件构成：

- **Loading.ets**：加载组件主入口，负责组件展示、生命周期管理和API封装
- **LoadingStyle.ets**：定义加载组件的样式接口，规范可配置属性
- **LottieLoading.ets**：基于Lottie实现的动画加载效果，提供默认动画展示

这种分层设计使组件具有良好的可维护性和扩展性，开发者可以通过样式接口轻松定制加载效果，而无需修改核心逻辑。

### 2. LoadingStyle接口定义

LoadingStyle接口定义了加载组件的可配置属性，位于`common/ui/src/main/ets/components/loading/LoadingStyle.ets`：

```typescript
export interface LoadingStyle {
  /** 位置，默认居中 */
  alignment?: Alignment
  /** 点击系统返回键可以取消吗，默认 true */
  pressBackCancel?: boolean
  /** 点击外面区域是否可以取消，默认 false */
  outsideCancel?: boolean
  /** 自定loading 动画布局 */
  layout?: WrappedBuilder<[object]>;
  /** 最外层UI的样式，设置背景深度等 */
  outBoxAttr?: AttributeModifier<StackAttribute>
}
```

接口设计遵循以下原则：
- 所有属性均为可选，确保组件有合理的默认行为
- 功能模块化，将布局、交互和样式配置分离
- 使用ArkUI的基础类型（如Alignment、AttributeModifier）确保兼容性
- 通过WrappedBuilder支持自定义布局，提供最大灵活性

### 3. Loading组件实现详解

#### 3.1 组件结构与生命周期

Loading组件采用NavDestination作为容器，以对话框模式展示，核心代码位于`common/ui/src/main/ets/components/loading/Loading.ets`：

```typescript
@ComponentV2
export struct Loading {
  private gStyle: LoadingStyle | undefined = global.getObject<LoadingStyle>('loadingStyle');
  private style: LoadingStyle = this.gStyle ? this.gStyle : {};
  private pathStack: NavPathStack = new NavPathStack()
  @Local backCancel: boolean = true;
  @Local outsideCancel: boolean = false;
  @Local alignment: Alignment = Alignment.Center;
  @Local layout: WrappedBuilder<[object]> = this.style.layout ? this.style.layout : wrapBuilder(defaultLoading);
  @Local outBoxAttr: AttributeModifier<StackAttribute> = {};
  @Provider() isStartLoading: boolean = false;

  build() {
    NavDestination() {
      Stack() {
        // 背景遮罩层
        Stack()
          .width('100%')
          .height('100%')
          .onClick(() => {
            if (this.outsideCancel) {
              this.pathStack.pop();
            }
          })
        // 加载动画布局
        this.layout.builder(undefined)
      }
      .backgroundColor($r('sys.color.mask_secondary'))
      .alignContent(this.alignment)
      .width('100%')
      .height('100%')
      .attributeModifier(this.outBoxAttr)
    }
    .onReady(cxt => {
      this.pathStack = cxt.pathStack
      const parm = cxt.pathInfo.param as LoadingStyle
      this.creatCurStyle(parm)
    })
    .onAppear(() => {
      this.isStartLoading = true;
    })
    .onDisAppear(() => {
      this.isStartLoading = false;
    })
    .onBackPressed(() => {
      return !this.backCancel;
    })
    .mode(NavDestinationMode.DIALOG)
    .hideTitleBar(true)
  }
  // ...其他方法
}
```

组件的核心特性包括：
- 使用NavDestinationMode.DIALOG模式，以对话框形式展示，不影响页面栈结构
- 通过Stack布局实现背景遮罩层和加载内容的层级关系
- 支持点击外部区域和返回键取消加载（可配置）
- 组件出现和消失时自动控制动画的开始和停止
- 通过@Provider装饰器与动画组件通信，控制动画状态

#### 3.2 API设计与使用

Loading组件提供了简洁的静态方法API，方便开发者在任何地方调用：

```typescript
/**
 * 全局初始化样式
 */
static initDefaultStyle(loadingStyle: LoadingStyle) {
  global.setObject('loadingStyle', loadingStyle)
}

public static open(parm: LoadingStyle = {}) {
  RouterUtil.push('Loading', parm);
}

public static close() {
  RouterUtil.finishPage('Loading');
}
```

这种设计的优势在于：
- 提供全局样式初始化，确保应用内加载组件风格统一
- 静态方法调用简单直观，无需创建组件实例
- 支持每次调用时传入不同配置，满足多样化场景需求
- 基于路由管理实现，与应用导航系统无缝集成

### 4. LottieLoading动画实现

默认加载动画基于Lottie实现，位于`common/ui/src/main/ets/components/loading/LottieLoading.ets`：

```typescript
@ComponentV2
export struct LottieLoading {
  private mainRenderingSettings: RenderingContextSettings = new RenderingContextSettings(true)
  private mainCanvasRenderingContext: CanvasRenderingContext2D = new CanvasRenderingContext2D(this.mainRenderingSettings)
  private jsonData: string = '';
  @Consumer() isStartLoading: boolean = false;

  @Monitor('isStartLoading')
  loadAnimation() {
    if (!this.isStartLoading) {
      lottie.destroy();
      return
    }
    this.createAnimation();
  }

  private createAnimation() {
    lottie.destroy();
    let resStr = new util.TextDecoder('utf-8', { ignoreBOM: true });
    let contexts = getContext() as common.UIAbilityContext
    contexts.resourceManager.getRawFile('loading.json', (err: Error, data: Uint8Array) => {
      if (data?.buffer) {
        let lottieStr = resStr.decode(new Uint8Array(data.buffer));
        this.jsonData = JSON.parse(lottieStr);

        lottie.loadAnimation({
          container: this.mainCanvasRenderingContext, // 渲染上下文
          renderer: 'canvas', // 渲染方式
          loop: true, // 循环播放
          autoplay: true, // 自动播放
          animationData: this.jsonData, // json动画数据
          context: contexts // 当前场景上下文
        })
      }
    })
  }

  build() {
    Canvas(this.mainCanvasRenderingContext)
      .width('85%')
      .height('50%')
      .onReady(() => {
        this.mainCanvasRenderingContext.imageSmoothingEnabled = true;
        this.mainCanvasRenderingContext.imageSmoothingQuality = 'medium'
      })
  }
}
```

Lottie动画实现的关键点：
- 使用Canvas组件作为渲染容器，通过RenderingContext2D绘制动画
- 监听isStartLoading状态变化，控制动画的启动与销毁
- 从应用资源中加载JSON格式的Lottie动画文件
- 支持抗锯齿设置，提升动画显示质量
- 动画结束或组件消失时销毁资源，避免内存泄漏

## 实用建议

### 1. 基础使用示例

```typescript
// 简单显示加载
Loading.open();

// 2秒后关闭加载
setTimeout(() => {
  Loading.close();
}, 2000);
```

### 2. 自定义配置示例

```typescript
// 自定义加载位置和取消行为
Loading.open({
  alignment: Alignment.Bottom,
  pressBackCancel: false,
  outsideCancel: true,
  outBoxAttr: {
    backgroundColor: Color.Black.withAlpha(0.7)
  }
});
```

### 3. 全局样式初始化

在应用入口处初始化全局加载样式：

```typescript
// app.ets或主页面初始化代码
Loading.initDefaultStyle({
  alignment: Alignment.Center,
  pressBackCancel: true,
  outsideCancel: false,
  // 设置全局默认加载动画
  layout: wrapBuilder(() => {
    Column() {
      Progress()
        .color(Color.Blue)
        .width(50)
        .height(50)
      Text('加载中...')
        .fontSize(14)
        .margin({ top: 10 })
    }
  })
});
```

### 4. 结合网络请求使用

与之前介绍的网络请求框架结合使用：

```typescript
// 发起请求前显示加载
Loading.open();

// 网络请求
net.post<DataResp>(API.getData, params, {
  onSuccess: (data) => {
    // 处理数据
  },
  onFail: (code, msg) => {
    // 显示错误信息
  },
  onFinally: () => {
    // 请求完成，关闭加载
    Loading.close();
  }
});
```

## 总结与展望

ZeroOneApp加载组件的基础实现具有以下特点：

1. **架构清晰**：采用模块化设计，将组件逻辑、样式定义和动画实现分离
2. **使用便捷**：提供简洁的静态API，支持一行代码调用
3. **配置灵活**：通过样式接口支持多种自定义需求
4. **性能优化**：合理管理动画资源，避免内存泄漏
5. **兼容性好**：基于HarmonyOS标准API开发，适配不同设备

基础使用场景已经覆盖了大多数加载需求，但在复杂应用中仍有扩展空间，如加载状态管理、多实例控制等高级特性将在后续文章中探讨。