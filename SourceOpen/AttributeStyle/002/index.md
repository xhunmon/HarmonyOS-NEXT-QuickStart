# AttributeStyle Style Composition Mechanism: Building a Flexible and Reusable UI Style System

## 1. Introduction

In complex UI development, a single component often needs to apply multiple styles (such as layout, color, interaction state, etc.). In traditional development methods, fragmented management of style code leads to maintenance difficulties. AttributeStyle, through its innovative style composition mechanism, allows developers to flexibly combine multiple independent style units, achieving modular reuse and precise control of styles, further improving code reusability and development efficiency.

## 2. Official Capability Foundation

HarmonyOS NEXT's custom extension modifier (AttributeModifier) supports developers in defining unified attribute modification logic for components, but the native capability does not provide a style combination and priority management mechanism<mcurl name="自定义扩展修饰器 AttributeModifier" url="https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-user-defined-extension-attributemodifier"></mcurl>. AttributeStyle has built a multi-level style composition architecture based on this capability, filling this gap.

## 3. Core Implementation of Style Composition

### 3.1 SAdapter: The Core Engine of Style Composition

As the adapter for style composition, SAdapter provides the `addStyle` method to实现有序叠加 of multiple style units, and generates an attribute modifier applicable to components through the `buildAttribute` method:

```typescript:library/src/main/ets/style/SAdapter.ets
addStyle(style: SStyle<T>): SAdapter<T> {
  if (this.styles === undefined) {
    this.styles = []
  }
  this.styles.push(style)
  return this
}

buildAttribute<R extends CommonInterface | undefined = undefined>(initLoadNormalStyle: boolean = true,
  disappearToNormalStyle: boolean = true): SAttribute<T, R> {
  return new SAttribute<T, R>(this, initLoadNormalStyle, disappearToNormalStyle)
}
```

**Key Features**:
- Supports chain calls to simplify multi-style composition code
- Maintains style execution order, with later added styles able to override同名属性 of earlier added styles
- Seamless integration with SAttribute to convert combined styles into component-recognizable modifiers

### 3.2 Multi-style Collaborative Execution Mechanism

SAdapter实现统一调度 of all combined styles through the `notifyAll` method, ensuring that all relevant styles are updated同步 when the state changes:

```typescript:library/src/main/ets/style/SAdapter.ets
private notifyAll(callback: (style: SStyle<T>) => void): void {
  this.styles?.forEach((style: SStyle<T>) => {
    try {
      callback(style)
    } catch (error) {
      Log.e(TAG, () => 'notifyAll', error)
    }
  })
}
```

Taking `pressedStyle` as an example, when a component triggers the pressed state, the pressing logic of all combined styles will be executed in sequence:

```typescript:library/src/main/ets/style/SAdapter.ets
pressedStyle(instance?: T | undefined): void {
  Log.d(TAG, () => 'pressedStyle...')
  this.notifyAll((style) => {
    style.pressedStyle?.(instance)
  })
}
```

## 4. State-Driven Style Scheduling: SController

As the state controller, SController is responsible for mapping component state changes to style execution instructions. Its core logic is as follows:

```typescript:library/src/main/ets/style/SController.ets
stateChange(state: SState, isAppear: boolean, instance?: T | undefined) {
  if (!isAppear && this.disappearToNormalStyle) { // Restore normal style when state disappears
    this.updateStateStyle(SState.Normal, instance)
    return
  }
  this.updateStateStyle(state, instance, isAppear)
}

private updateStateStyle(state: SState, instance?: T | undefined, appear?: boolean) {
  Log.d(TAG, () => 'updateStateStyle state: ' + state)
  switch (state) {
    case SState.Normal: this.adapter.normalStyle(instance); break
    case SState.Pressed: this.adapter.pressedStyle(instance); break
    // Other state handling...
  }
}
```

**Workflow**:
1. Receive state change events (such as press, hover, select)
2. Call the corresponding style method in SAdapter according to the state type
3. Can automatically restore normal style when the state disappears (configured via disappearToNormalStyle)

## 5. Practical Application Case

In the Index page, 5 independent styles are combined through SAdapter to build complex button effects:

```typescript:entry/src/main/ets/pages/Index.ets
private style1 = new SAdapter<TextAttribute>()
  .addStyle(new ContainStyle()) // Layout style
  .addStyle(new BgGreenStyle()) // Background style
  .addStyle(new TextNormalStyle()) // Text default style
  .addStyle(new TextPressedStyle()) // Text pressed style
  .addStyle(new HoverBgStyle()) // Hover background style
  .buildAttribute(true, true)
```

**Advantage Analysis**:
- **Modularity**: Each style is maintained independently, such as ContainStyle focusing on layout control
- **Reusability**: The same style combination can be shared among multiple components
- **Extensibility**: New styles only need to implement the SStyle interface and be added via addStyle

## 6. Summary and Outlook

AttributeStyle's style composition mechanism solves the problem of code fragmentation in traditional style management through the collaborative work of SAdapter and SController. Future plans include introducing a style priority weight system and conditional style rules to further enhance the flexibility of style composition.

## 7. Reference Links
- Project Address: <mcurl name="AttributeStyle" url="https://gitee.com/qincji/AttributeStyle"></mcurl>
- Official Documentation: <mcurl name="自定义扩展属性更新器 AttributeUpdater" url="https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-user-defined-extension-attributeupdater"></mcurl>