# AttributeStyle Core Technology Analysis: UI Style Reuse Solution Based on HarmonyOS NEXT

## 1. Introduction

In the HarmonyOS application development process, repeated definition and state management of UI styles have always been pain points affecting development efficiency. In traditional approaches, developers need to write a lot of similar style code repeatedly for different components and states, which not only increases maintenance costs but also leads to code redundancy. Based on the custom modifier capabilities of HarmonyOS NEXT 5.0.0, the AttributeStyle project proposes an innovative UI style reuse solution that can reduce 70% of repetitive code and significantly improve development efficiency.

## 2. Official Capability Introduction

HarmonyOS NEXT provides two core capabilities: AttributeModifier and AttributeUpdater, supporting developers to customize component attributes and state update logic:

- **AttributeModifier**: Allows developers to customize component attribute modifiers for unified management and reuse of attributes<mcurl name="自定义修饰器 AttributeModifier" url="https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-user-defined-modifier#attributemodifier"></mcurl>
- **AttributeUpdater**: Provides a callback mechanism for component state changes, enabling dynamic update of component styles based on different states<mcurl name="自定义扩展属性更新器 AttributeUpdater" url="https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-user-defined-extension-attributeupdater"></mcurl>

## 3. AttributeStyle Design and Implementation

### 3.1 Core Architecture

AttributeStyle adopts a layered design, mainly including the following core modules:

- **SAttribute**: Inherits from AttributeUpdater, serving as the base class for style management
- **SAdapter**: Style adapter responsible for managing the combination and priority of multiple styles
- **SController**: State controller that handles state change logic and applies corresponding styles
- **SStyle**: Style interface defining style application rules under different states

### 3.2 Key Implementation

As the core base class, SAttribute implements state monitoring by overriding the callback methods of AttributeUpdater:

```typescript:library/src/main/ets/style/SAttribute.ets
// System API callback for press event
applyPressedAttribute(instance: T): void {
  Log.d(TAG, () => 'applyPressedAttribute...')
  this.controller.stateChange(SState.Pressed, true, instance)
}

// System API callback for disabled state
applyDisabledAttribute(instance: T): void {
  Log.d(TAG, () => 'applyDisabledAttribute...')
  this.controller.stateChange(SState.Disabled, true, instance)
}
```

SAdapter implements the style combination mechanism, allowing multiple styles to be applied in叠加:

```typescript:entry/src/main/ets/pages/Index.ets
private style1 = new SAdapter<TextAttribute>()
  .addStyle(new ContainStyle())
  .addStyle(new BgGreenStyle())
  .addStyle(new TextNormalStyle())
  .addStyle(new TextPressedStyle())
  .addStyle(new HoverBgStyle())
  .buildAttribute(true, true)
```

## 4. Application Effects and Advantages

### 4.1 Improved Code Reusability

With AttributeStyle, a complex style can be reused by multiple components. For example, in the Index page, the combined style defined by style1 is applied to multiple Text components, avoiding writing the same style code repeatedly.

### 4.2 Simplified State Management

Traditional approaches require manual monitoring and handling of various state changes, while AttributeStyle uniformly manages states through SController and automatically applies corresponding styles:

```typescript:entry/src/main/ets/pages/Index.ets
Toggle({ type: ToggleType.Switch })
  .onChange((isOn) => {
    this.style5.updateSelect(isOn)
  })
```

## 5. Summary and Outlook

Based on the custom modifier capabilities of HarmonyOS NEXT, AttributeStyle effectively solves the problem of code redundancy in UI development through innovative style combination and state management mechanisms. Future plans include adding more preset style libraries and optimizing the style priority algorithm to further enhance the development experience.

## 6. Reference Links
- Project Address: <mcurl name="AttributeStyle" url="https://gitee.com/qincji/AttributeStyle"></mcurl>
- Official Documentation: <mcurl name="自定义扩展修饰器 AttributeModifier" url="https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-user-defined-extension-attributemodifier"></mcurl>