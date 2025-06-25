# ZeroOneApp Loading Component: Basic Implementation and Usage

## Introduction

In mobile application development, displaying loading states is a crucial aspect of enhancing user experience. A well-designed loading component can effectively alleviate user waiting anxiety and provide clear interaction feedback. ZeroOneApp's UI component library includes a fully functional loading component developed based on HarmonyOS's ArkUI framework, supporting multiple loading styles, custom configurations, and convenient API calls. This article will analyze the implementation principles of ZeroOneApp's loading component from a basic level, including its core architecture, style definitions, and basic usage methods, helping developers quickly master the integration and application of this component.

## Official Resources

- **ArkUI Component Development Guide**: Official HarmonyOS documentation providing UI component development specifications and best practices [ArkUI Component Development Documentation](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkui)
- **Lottie Animation Library**: Airbnb's open-source animation library that supports rendering JSON format animation files [Lottie Official Documentation](https://lottiefiles.com/developers)
- **HarmonyOS Lottie Adaptation Library**: Lottie animation implementation for HarmonyOS [@ohos/lottie](https://ohpm.openharmony.cn/#/cn/detail/@ohos%2Flottie)

## Detailed Explanation

### 1. Core Architecture of Loading Component

ZeroOneApp's loading component adopts a modular design, mainly consisting of three core files:

- **Loading.ets**: Main entry of the loading component, responsible for component display, lifecycle management, and API encapsulation
- **LoadingStyle.ets**: Defines the style interface of the loading component, standardizing configurable properties
- **LottieLoading.ets**: Animation loading effect implemented based on Lottie, providing default animation display

This layered design gives the component good maintainability and extensibility. Developers can easily customize loading effects through the style interface without modifying core logic.

### 2. LoadingStyle Interface Definition

The LoadingStyle interface defines the configurable properties of the loading component, located in `common/ui/src/main/ets/components/loading/LoadingStyle.ets`:

```typescript
export interface LoadingStyle {
  /** Position, default is center */
  alignment?: Alignment
  /** Whether pressing the system back button can cancel, default true */
  pressBackCancel?: boolean
  /** Whether clicking outside area can cancel, default false */
  outsideCancel?: boolean
  /** Custom loading animation layout */
  layout?: WrappedBuilder<[object]>;
  /** Style of the outermost UI, for setting background depth, etc. */
  outBoxAttr?: AttributeModifier<StackAttribute>
}
```

The interface design follows these principles:
- All properties are optional to ensure the component has reasonable default behavior
- Functional modularization, separating layout, interaction, and style configurations
- Using ArkUI's basic types (such as Alignment, AttributeModifier) to ensure compatibility
- Supporting custom layouts through WrappedBuilder for maximum flexibility

### 3. Detailed Implementation of Loading Component

#### 3.1 Component Structure and Lifecycle

The Loading component uses NavDestination as a container and displays in dialog mode. The core code is located in `common/ui/src/main/ets/components/loading/Loading.ets`:

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
        // Background mask layer
        Stack()
          .width('100%')
          .height('100%')
          .onClick(() => {
            if (this.outsideCancel) {
              this.pathStack.pop();
            }
          })
        // Loading animation layout
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
  // ...other methods
}
```

Core features of the component include:
- Using NavDestinationMode.DIALOG mode to display as a dialog without affecting the page stack structure
- Implementing hierarchical relationship between background mask layer and loading content through Stack layout
- Supporting cancellation of loading by clicking outside area and pressing back button (configurable)
- Automatically controlling animation start and stop when the component appears and disappears
- Communicating with animation components through @Provider decorator to control animation state

#### 3.2 API Design and Usage

The Loading component provides a concise static method API for easy calling anywhere by developers:

```typescript
/**
 * Global initialization of style
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

Advantages of this design:
- Providing global style initialization to ensure consistent style of loading components within the application
- Simple and intuitive static method calls without creating component instances
- Supporting passing different configurations for each call to meet diverse scenario requirements
- Implemented based on route management, seamlessly integrating with application navigation system

### 4. LottieLoading Animation Implementation

The default loading animation is implemented based on Lottie, located in `common/ui/src/main/ets/components/loading/LottieLoading.ets`:

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
          container: this.mainCanvasRenderingContext, // Rendering context
          renderer: 'canvas', // Rendering method
          loop: true, // Loop playback
          autoplay: true, // Auto play
          animationData: this.jsonData, // JSON animation data
          context: contexts // Current scene context
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

Key points of Lottie animation implementation:
- Using Canvas component as rendering container, drawing animation through RenderingContext2D
- Monitoring isStartLoading state changes to control animation start and destruction
- Loading JSON format Lottie animation files from application resources
- Supporting anti-aliasing settings to improve animation display quality
- Destroying resources when animation ends or component disappears to avoid memory leaks

## Practical Recommendations

### 1. Basic Usage Example

```typescript
// Simply display loading
Loading.open();

// Close loading after 2 seconds
setTimeout(() => {
  Loading.close();
}, 2000);
```

### 2. Custom Configuration Example

```typescript
// Custom loading position and cancellation behavior
Loading.open({
  alignment: Alignment.Bottom,
  pressBackCancel: false,
  outsideCancel: true,
  outBoxAttr: {
    backgroundColor: Color.Black.withAlpha(0.7)
  }
});
```

### 3. Global Style Initialization

Initialize global loading style at application entry:

```typescript
// Initialization code in app.ets or main page
Loading.initDefaultStyle({
  alignment: Alignment.Center,
  pressBackCancel: true,
  outsideCancel: false,
  // Set global default loading animation
  layout: wrapBuilder(() => {
    Column() {
      Progress()
        .color(Color.Blue)
        .width(50)
        .height(50)
      Text('Loading...')
        .fontSize(14)
        .margin({ top: 10 })
    }
  })
});
```

### 4. Using with Network Requests

Using with the previously introduced network request framework:

```typescript
// Show loading before发起请求
Loading.open();

// Network request
net.post<DataResp>(API.getData, params, {
  onSuccess: (data) => {
    // Process data
  },
  onFail: (code, msg) => {
    // Show error message
  },
  onFinally: () => {
    // Request completed, close loading
    Loading.close();
  }
});
```

## Summary and Outlook

The basic implementation of ZeroOneApp's loading component has the following characteristics:

1. **Clear Architecture**: Adopting modular design, separating component logic, style definition, and animation implementation
2. **Easy to Use**: Providing concise static API, supporting one-line code calling
3. **Flexible Configuration**: Supporting multiple custom requirements through style interface
4. **Performance Optimization**: Reasonably managing animation resources to avoid memory leaks
5. **Good Compatibility**: Developed based on HarmonyOS standard API, adapting to different devices

Basic usage scenarios already cover most loading requirements, but there is still room for expansion in complex applications. Advanced features such as loading state management and multi-instance control will be discussed in subsequent articles.