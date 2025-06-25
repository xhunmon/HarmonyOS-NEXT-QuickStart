# ZeroOneApp Loading Component: Advanced Customization and Best Practices

## Introduction

As a crucial feedback mechanism for user interaction, the quality of loading components directly impacts the user experience of an application. Building upon its basic implementation, ZeroOneApp's loading component offers rich advanced customization capabilities and performance optimization strategies, meeting diverse requirements in complex application scenarios. This article will explore advanced customization techniques, performance optimization practices, and integration solutions with other system features for the loading component, helping developers create smoother, more personalized loading experiences.

## Official Resources

- **ArkUI Custom Component Development Guide**: Detailed instructions on building reusable, highly extensible custom components [ArkUI Custom Component Documentation](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkui)
- **HarmonyOS Performance Optimization Guide**: Official recommendations for application performance tuning [HarmonyOS Performance Optimization Documentation](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-high-performance-programming)
- **Lottie Animation Advanced Features**: Advanced usage and performance optimization techniques for Lottie animations [Lottie Advanced Usage](https://lottiefiles.com/blog/developers/working-with-lottie/lottie-animation-advanced-tips/)

## Detailed Explanation

### 1. Advanced Style Customization Techniques

#### 1.1 Complete Guide to Custom Loading Layouts

ZeroOneApp's loading component supports fully customized loading layouts through the `layout` property, implemented using ArkUI's `WrappedBuilder`. Here's an example of a complex custom layout:

```typescript
// Custom loading layout example
const CustomLoadingLayout = wrapBuilder((message: string) => {
  Column() {
    // Custom animation component
    Image($r('app.media.loading_icon'))
      .width(60)
      .height(60)
      .animation({
        duration: 1000,
        curve: Curve.Linear,
        iterations: -1,
        playMode: PlayMode.Normal
      })
      .rotate({ angle: 360 })

    // Loading text
    Text(message || 'Loading data...')
      .fontSize(16)
      .fontColor($r('app.color.primary_text'))
      .margin({ top: 12 })

    // Cancel button
    Button('Cancel')
      .width(120)
      .height(36)
      .margin({ top: 20 })
      .onClick(() => {
        Loading.close();
      })
  }
  .padding(24)
  .backgroundColor(Color.White)
  .borderRadius(16)
  .shadow({
    radius: 10,
    color: Color.Black.withAlpha(0.1),
    offsetX: 0,
    offsetY: 4
  })
});

// Using custom layout
Loading.open({
  layout: CustomLoadingLayout,
  alignment: Alignment.Center,
  outsideCancel: true
});
```

Key technical points for custom layouts:
- Wrap any UI structure with `wrapBuilder` to achieve complete layout customization
- Combine with ArkUI's animation system to implement non-Lottie animation effects
- Add interactive elements (like cancel buttons) to enhance user control
- Beautify appearance with properties like padding, backgroundColor, and borderRadius
- Add depth with shadow properties to improve visual experience

#### 1.2 Global vs. Local Style Priority Mechanism

ZeroOneApp's loading component supports both global default styles and local call styles, following clear priority rules:

```typescript
// 1. Global style initialization (at application startup)
Loading.initDefaultStyle({
  alignment: Alignment.Center,
  pressBackCancel: true,
  outsideCancel: false,
  outBoxAttr: {
    backgroundColor: Color.Black.withAlpha(0.5)
  }
});

// 2. Override specific styles when calling locally
Loading.open({
  // Only override properties that need modification, others use global configuration
  outsideCancel: true,
  alignment: Alignment.Bottom
});
```

Priority order (from highest to lowest):
1. `LoadingStyle` parameters passed during local calls
2. Globally initialized `loadingStyle` configuration
3. Component internal default values

This mechanism ensures style consistency within the application while allowing flexible adjustments for specific scenarios.

### 2. Advanced Lottie Animation Applications

#### 2.1 Multi-animation Resource Management and Dynamic Switching

For complex applications requiring multiple loading animations, implement dynamic management of animation resources:

```typescript
// Animation resource management utility class
class LottieAnimationManager {
  private static animations: Map<string, string> = new Map();

  // Preload animation resources
  static preloadAnimations(context: common.UIAbilityContext) {
    const animationFiles = ['loading_default.json', 'loading_success.json', 'loading_error.json'];
    const decoder = new util.TextDecoder('utf-8', { ignoreBOM: true });

    animationFiles.forEach(file => {
      context.resourceManager.getRawFile(file, (err, data) => {
        if (data?.buffer) {
          const jsonStr = decoder.decode(new Uint8Array(data.buffer));
          this.animations.set(file, jsonStr);
        }
      });
    });
  }

  // Get animation data
  static getAnimationData(key: string): object | undefined {
    const jsonStr = this.animations.get(key);
    return jsonStr ? JSON.parse(jsonStr) : undefined;
  }
}

// Custom Lottie loading component
@ComponentV2
struct CustomLottieLoading {
  @Prop animationKey: string = 'loading_default.json';
  // ...other properties and methods

  private createAnimation() {
    lottie.destroy();
    const animationData = LottieAnimationManager.getAnimationData(this.animationKey);
    if (animationData) {
      lottie.loadAnimation({
        container: this.mainCanvasRenderingContext,
        renderer: 'canvas',
        loop: this.animationKey === 'loading_default.json', // Only default animation loops
        autoplay: true,
        animationData: animationData,
        context: getContext() as common.UIAbilityContext
      });
    }
  }
}
```

Advantages of this implementation:
- Preloading mechanism reduces animation display delay
- Centralized management of multiple animation resources for easier maintenance
- Supports dynamic switching between animations for different states (loading, success, error)
- Non-looping animations for status prompts enhance interaction experience

#### 2.2 Lottie Animation Performance Optimization

For performance issues that may arise with Lottie animations, implement the following optimization strategies in `common/ui/src/main/ets/components/loading/LottieLoading.ets`:

```typescript
// Optimized LottieLoading component
@ComponentV2
export struct OptimizedLottieLoading {
  // ...other properties
  private animationInstance: any = null;
  private isAnimationLoaded: boolean = false;

  // Limit animation frame rate
  private frameRate: number = 30;
  private lastFrameTime: number = 0;

  @Monitor('isStartLoading')
  loadAnimation() {
    if (!this.isStartLoading) {
      this.destroyAnimation();
      return;
    }
    // Play directly if animation is already loaded to avoid re-creation
    if (this.isAnimationLoaded && this.animationInstance) {
      this.animationInstance.play();
      return;
    }
    this.createAnimation();
  }

  private createAnimation() {
    // ...original code
    this.animationInstance = lottie.loadAnimation({
      // ...configuration
      rendererSettings: {
        preserveAspectRatio: 'xMidYMid meet',
        clearCanvas: true
      }
    });
    this.isAnimationLoaded = true;

    // Custom frame rate control
    this.animationInstance.setSubframe(false);
    this.animationInstance.frameRate = this.frameRate;
  }

  private destroyAnimation() {
    if (this.animationInstance) {
      this.animationInstance.stop();
      // Keep instance but stop playback to avoid frequent destruction and creation
      // lottie.destroy()
    }
  }

  build() {
    Canvas(this.mainCanvasRenderingContext)
      .width('85%')
      .height('50%')
      .onDisappear(() => {
        this.destroyAnimation();
      })
  }
}
```

Key optimization points:
- Avoid frequent destruction and creation of animation instances by using pause/play mode
- Limit frame rate (e.g., 30fps) to reduce CPU usage
- Disable subframe rendering (setSubframe(false)) to reduce drawing complexity
- Properly set preserveAspectRatio to avoid unnecessary scaling calculations
- Stop animations promptly when components disappear to release resources

### 3. Advanced Interaction and State Management

#### 3.1 Decoupling Loading State from Business Logic

Implement decoupling of loading state and business logic through state management patterns:

```typescript
// Loading state management class
class LoadingManager {
  private static loadingCount: number = 0;

  // Show loading with support for叠加计数
  static showLoading(options: LoadingStyle = {}): void {
    if (this.loadingCount === 0) {
      Loading.open(options);
    }
    this.loadingCount++;
  }

  // Hide loading, requires matching show count
  static hideLoading(): void {
    if (this.loadingCount > 0) {
      this.loadingCount--;
      if (this.loadingCount === 0) {
        Loading.close();
      }
    }
  }

  // Force hide all loading
  static forceHide(): void {
    this.loadingCount = 0;
    Loading.close();
  }
}

// Usage in business logic
class DataService {
  fetchData(): Promise<any> {
    LoadingManager.showLoading();
    return new Promise((resolve) => {
      // Network request
      net.post<DataResp>(API.data, {}, {
        onSuccess: (data) => resolve(data),
        onFail: () => resolve(null),
        onFinally: () => LoadingManager.hideLoading()
      });
    });
  }
}
```

This design solves the following problems:
- Loading state conflicts with multiple concurrent requests
- Avoid loading flicker caused by rapid consecutive calls
- Unified management of all loading states in the application
- Simplified loading control logic in business code

#### 3.2 Integration with Routing and Page Lifecycle

Implement automatic association of loading state with page lifecycle:

```typescript
// Integrate loading management in route interceptor
RouterUtil.addInterceptor({
  beforePush: (url: string) => {
    // Some pages need automatic loading display
    if (['DetailPage', 'ProfilePage'].includes(url)) {
      LoadingManager.showLoading();
    }
    return true;
  },
  afterPush: (url: string) => {
    // Automatically hide loading after page entry
    if (['DetailPage', 'ProfilePage'].includes(url)) {
      // Allow time for page initialization
      setTimeout(() => LoadingManager.hideLoading(), 300);
    }
  }
});

// Usage in page components
@Component
struct DetailPage {
  aboutToAppear() {
    // Start data loading when page appears
    this.loadData();
  }

  loadData() {
    // Loading already shown by route interceptor, no need to call again
    dataService.fetchDetail().then(data => {
      // Process data
    });
  }
}
```

### 4. Special Scenario Handling

#### 4.1 Loading Strategies for Weak Network Environments

Optimize loading experience for weak network environments:

```typescript
// Weak network detection utility
class NetworkQualityMonitor {
  static isWeakNetwork(): boolean {
    const netType = network.getCurrentType();
    const signal = network.getSignalStrength();
    // Determine based on network type and signal strength
    return netType === network.NetworkType.NETWORK_MOBILE && signal < 2 ||
           netType === network.NetworkType.NETWORK_WIFI && signal < 30;
  }
}

// Weak network loading configuration
const createWeakNetworkLoading = () => {
  return wrapBuilder(() => {
    Column() {
      LottieLoading()
      Text(NetworkQualityMonitor.isWeakNetwork() ? 'Poor network condition, still trying to load...' : 'Loading...')
        .fontSize(14)
        .margin({ top: 10 })
      if (NetworkQualityMonitor.isWeakNetwork()) {
        Button('Switch to Offline Mode')
          .margin({ top: 15 })
          .onClick(() => {
            AppState.switchOfflineMode();
            Loading.close();
          })
      }
    }
  });
};

// Use weak network adapted loading
Loading.open({
  layout: createWeakNetworkLoading(),
  pressBackCancel: true
});
```

#### 4.2 Loading Timeout Handling

Implement automatic cancellation mechanism for loading timeouts:

```typescript
// Loading utility with timeout
class TimeoutLoading {
  private timeoutId: number = -1;

  showLoading(options: LoadingStyle = {}, timeoutMs: number = 10000): Promise<boolean> {
    return new Promise((resolve) => {
      Loading.open(options);
      // Set timeout timer
      this.timeoutId = setTimeout(() => {
        Loading.close();
        resolve(false); // Timeout
      }, timeoutMs);

      // Listen for loading close event (implemented through custom events)
      LoadingEvent.on('close', () => {
        if (this.timeoutId !== -1) {
          clearTimeout(this.timeoutId);
          resolve(true); // Normal close
        }
      });
    });
  }
}

// Usage example
const timeoutLoading = new TimeoutLoading();
const loadSuccess = await timeoutLoading.showLoading({}, 8000);
if (!loadSuccess) {
  promptAction.showToast({ message: 'Loading timed out, please try again' });
}
```

## Practical Recommendations

### 1. Performance Optimization Checklist

- **Animation Optimization**:
  - Prefer Lottie for complex animations over custom property animations
  - Control animation frame rate below 30fps
  - Avoid playing multiple animations simultaneously

- **Resource Management**:
  - Preload commonly used animation resources
  - Destroy animation instances when no longer needed
  - Compress Lottie JSON file sizes

- **Interaction Design**:
  - All loading states must provide a cancellation method
  - Long loading processes should display progress hints
  - Error states should have clear retry guidance

### 2. Accessibility Optimization

Ensure loading components are accessible to all users:

```typescript
// Accessible loading component
@Builder
function AccessibleLoading() {
  Column() {
    LottieLoading()
    Text('Loading data')
      .fontSize(16)
      .margin({ top: 12 })
  }
  .semanticsLabel('Data loading') // Accessibility label
  .semanticsHint('Please wait, data is being loaded') // Accessibility hint
  .focusable(true) // Support focus
}
```

### 3. Testing Strategy

- **Unit Testing**: Test the counting logic of LoadingManager
- **Performance Testing**: Monitor animation CPU usage with performance analysis tools
- **Scenario Testing**: Simulate weak network, timeout, concurrent requests, etc.
- **UI Testing**: Verify loading positions and styles on different sized devices

## Summary and Outlook

The advanced features of ZeroOneApp's loading component provide powerful support for complex application scenarios, with key advantages including:

1. **Highly Customizable**: From simple style adjustments to fully custom layouts, meeting diverse design needs
2. **Performance Optimized**: Ensuring smooth experiences through animation management and resource preloading
3. **Scenario Adaptable**: Providing solutions for special cases like weak networks and timeouts
4. **Architecturally Flexible**: Supporting deep integration with state management and routing systems

Future exploration directions:
- Progressive loading solutions based on skeleton screens
- Intelligent loading strategies combined with application state
- Consistent loading experiences across devices
- A/B testing framework for loading states