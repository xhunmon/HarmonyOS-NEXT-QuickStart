# AttributeStyle State Management System: Building Responsive UI Interaction Experiences

## 1. Introduction

State management of UI components is a core环节 in building interactive experiences. In HarmonyOS application development, the native state callback mechanism is scattered across various component methods, leading to high coupling between state logic and UI styles. AttributeStyle decouples component states (such as pressed, hovered, selected, error, etc.) from style logic through a unified state management system,实现自动化响应 of state changes and style reuse, significantly improving interactive development efficiency.

## 2. Official State Capability Foundation

HarmonyOS provides basic component state callbacks such as `onHover` and `onFocus`, but these callbacks need to be implemented individually in each component and lack a unified state management mechanism<mcurl name="自定义扩展属性更新器 AttributeUpdater" url="https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-user-defined-extension-attributeupdater"></mcurl>. AttributeStyle extends state types and builds a standardized state-style mapping system based on native capabilities.

## 3. Core Design of State Management

### 3.1 SState: Standardized State Enumeration

SState defines a complete set of component states, adding commonly used business states to the native states:

```typescript:library/src/main/ets/style/SState.ets
export enum SState {
  Init = 0, // Initialization (triggered only once)
  Normal = 1, // Normal state
  Pressed = 2, // Pressed
  Disabled = 3, // Disabled
  Hovered = 4, // Mouse hover
  Focused = 5, // Focused
  Selected = 6, // [Extended] Selected state
  Error = 7, // [Extended] Error state
  Custom = 8 // [Extended] Custom state
}
```

**Extended Value**:
- **Selected**: Supports checkbox, tab page and other selected state management
- **Error**: Unified form validation error style handling
- **Custom**: Allows developers to define business-specific states (such as loading, success, etc.)

### 3.2 SIState: State-Style Mapping Interface

The SIState interface defines style methods corresponding to each state,实现契约式绑定 of state and style logic:

```typescript:library/src/main/ets/style/SIState.ets
export interface SIState<T> {
  normalStyle?: (instance?: T) => void;
  pressedStyle?: (instance?: T) => void;
  disabledStyle?: (instance?: T) => void;
  hoverStyle?: (instance?: T, appear?: boolean) => void;
  focusStyle?: (instance?: T, appear?: boolean) => void;
  selectedStyle?: (instance?: T, appear?: boolean) => void;
  errorStyle?: (instance?: T, appear?: boolean) => void;
}
```

**Design Features**:
- Each state corresponds to an independent method with a single responsibility
- Optional implementation mechanism, only need to define the states required by the component
- Unified parameter specification, facilitating batch scheduling by SAdapter

## 4. State Flow and Style Application

### 4.1 State Monitoring and Forwarding

SAttribute serves as the state entry point,监听原生组件 events and converting them into SState states:

```typescript:library/src/main/ets/style/SAttribute.ets
instance?.onHover((isHover: boolean) => {
  this.controller.stateChange(SState.Hovered, isHover, this.attribute)
})
.onFocus(() => {
  this.controller.stateChange(SState.Focused, true, this.attribute)
})
.onBlur(() => {
  this.controller.stateChange(SState.Focused, false, this.attribute)
})
```

### 4.2 State-Driven Style Execution

SController calls the corresponding style method according to the state type,实现自动映射 from state to style:

```typescript:library/src/main/ets/style/SController.ets
private updateStateStyle(state: SState, instance?: T | undefined, appear?: boolean) {
  switch (state) {
    case SState.Normal: this.adapter.normalStyle(instance); break
    case SState.Pressed: this.adapter.pressedStyle(instance); break
    case SState.Hovered: this.adapter.hoverStyle(instance, appear); break
    case SState.Selected: this.adapter.selectedStyle(instance, appear); break
    // Other state handling...
  }
}
```

## 5. Practical Application: Multi-State Component Development

### 5.1 Selected State Example

Switch the selected state through the `updateSelect` method,自动应用 selectedStyle:

```typescript:entry/src/main/ets/pages/Index.ets
private style5 = new SAdapter<TextAttribute>()
  .addStyle(new ContainStyle())
  .addStyle({ normalStyle: instance => instance?.backgroundColor('#ff858282') })
  .addStyle(new SelectedBgStyle()) // Selected style
  .buildAttribute(true, true)

// Toggle switch controls selected state
Toggle({ type: ToggleType.Switch })
  .onChange((isOn) => this.style5.updateSelect(isOn))
```

### 5.2 Error State Example

When form validation fails, trigger error style through `updateError`:

```typescript:entry/src/main/ets/pages/Index.ets
private style6 = new SAdapter<TextAttribute>()
  .addStyle(new ContainStyle())
  .addStyle({ normalStyle: instance => instance?.backgroundColor('#fff58787') })
  .addStyle(new ErrorBorderStyle()) // Error style
  .buildAttribute(true, false)

// Switch controls error state
Toggle({ type: ToggleType.Switch })
  .onChange((isOn) => this.style6.updateError(isOn))
```

## 6. Technical Advantage Analysis

### 6.1 Centralized State Logic

Unified management of scattered state callbacks,避免组件代码中充斥状态处理逻辑

### 6.2 Maximized Style Reuse

The same state style (such as error borders) can be reused across multiple components, reducing duplicate code

### 6.3 Flexible Business State Extension

Support business-specific states through Custom state, such as:
```typescript
// Custom loading state style
const loadingStyle: SStyle<TextAttribute> = {
  customStyle: (instance, appear) => {
    if (appear) instance?.backgroundColor('#ffcccccc').text('Loading...')
    else instance?.backgroundColor('#ffffffff').text('Loaded')
  }
}
```

## 7. Summary and Outlook

AttributeStyle's state management system solves the problem of state-style coupling in traditional development through standardized state definitions,契约式样式接口 and automated state flow. Future plans include introducing state combinations (such as handling Focus+Hover simultaneously) and state animation transition capabilities to further enhance interactive experiences.

## 8. Reference Links
- Project Address: <mcurl name="AttributeStyle" url="https://gitee.com/qincji/AttributeStyle"></mcurl>
- Official Documentation: <mcurl name="自定义修饰器 AttributeModifier" url="https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-user-defined-modifier#attributemodifier"></mcurl>