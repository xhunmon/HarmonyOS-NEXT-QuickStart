# AttributeStyle状态管理系统：构建响应式UI交互体验

## 1. 引言

UI组件的状态管理是构建交互体验的核心环节。HarmonyOS应用开发中，原生状态回调机制分散在各个组件方法中，导致状态逻辑与UI样式高度耦合。AttributeStyle通过统一的状态管理系统，将组件状态（如按压、悬停、选中、错误等）与样式逻辑解耦，实现了状态变化的自动化响应与样式复用，显著提升了交互开发效率。

## 2. 官方状态能力基础

HarmonyOS提供了基础的组件状态回调，如`onHover`、`onFocus`等，但这些回调需要在每个组件中单独实现，且缺乏统一的状态管理机制<mcurl name="自定义扩展属性更新器 AttributeUpdater" url="https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-user-defined-extension-attributeupdater"></mcurl>。AttributeStyle在原生能力基础上，扩展了状态类型并构建了标准化的状态-样式映射体系。

## 3. 状态管理核心设计

### 3.1 SState：标准化状态枚举

SState定义了组件的完整状态集合，在原生状态基础上新增了业务常用状态：

```typescript:library/src/main/ets/style/SState.ets
export enum SState {
  Init = 0, // 初始化（仅触发一次）
  Normal = 1, // 正常状态
  Pressed = 2, // 按下
  Disabled = 3, // 不可用
  Hovered = 4, // 鼠标悬停
  Focused = 5, // 获取焦点
  Selected = 6, // 【扩展】选中状态
  Error = 7, // 【扩展】错误状态
  Custom = 8 // 【扩展】自定义状态
}
```

**扩展价值**：
- **Selected**：支持复选框、标签页等选中状态管理
- **Error**：统一表单验证错误样式处理
- **Custom**：允许开发者定义业务特定状态（如加载中、成功等）

### 3.2 SIState：状态-样式映射接口

SIState接口定义了每种状态对应的样式方法，实现状态与样式逻辑的契约式绑定：

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

**设计特点**：
- 每种状态对应独立方法，职责单一
- 可选实现机制，仅需定义组件所需状态
- 统一参数规范，便于SAdapter批量调度

## 4. 状态流转与样式应用

### 4.1 状态监听与转发

SAttribute作为状态入口，监听原生组件事件并转换为SState状态：

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

### 4.2 状态驱动的样式执行

SController根据状态类型调用对应样式方法，实现状态到样式的自动映射：

```typescript:library/src/main/ets/style/SController.ets
private updateStateStyle(state: SState, instance?: T | undefined, appear?: boolean) {
  switch (state) {
    case SState.Normal: this.adapter.normalStyle(instance); break
    case SState.Pressed: this.adapter.pressedStyle(instance); break
    case SState.Hovered: this.adapter.hoverStyle(instance, appear); break
    case SState.Selected: this.adapter.selectedStyle(instance, appear); break
    // 其他状态处理...
  }
}
```

## 5. 实战应用：多状态组件开发

### 5.1 选中状态示例

通过`updateSelect`方法切换选中状态，自动应用selectedStyle：

```typescript:entry/src/main/ets/pages/Index.ets
private style5 = new SAdapter<TextAttribute>()
  .addStyle(new ContainStyle())
  .addStyle({ normalStyle: instance => instance?.backgroundColor('#ff858282') })
  .addStyle(new SelectedBgStyle()) // 选中样式
  .buildAttribute(true, true)

// 切换开关控制选中状态
Toggle({ type: ToggleType.Switch })
  .onChange((isOn) => this.style5.updateSelect(isOn))
```

### 5.2 错误状态示例

表单验证失败时，通过`updateError`触发错误样式：

```typescript:entry/src/main/ets/pages/Index.ets
private style6 = new SAdapter<TextAttribute>()
  .addStyle(new ContainStyle())
  .addStyle({ normalStyle: instance => instance?.backgroundColor('#fff58787') })
  .addStyle(new ErrorBorderStyle()) // 错误样式
  .buildAttribute(true, false)

// 开关控制错误状态
Toggle({ type: ToggleType.Switch })
  .onChange((isOn) => this.style6.updateError(isOn))
```

## 6. 技术优势分析

### 6.1 状态逻辑集中化
将分散的状态回调统一管理，避免组件代码中充斥状态处理逻辑

### 6.2 样式复用最大化
同一状态样式（如错误边框）可在多个组件间复用，减少重复代码

### 6.3 业务状态扩展灵活
通过Custom状态支持业务特定状态，如：
```typescript
// 自定义加载状态样式
const loadingStyle: SStyle<TextAttribute> = {
  customStyle: (instance, appear) => {
    if (appear) instance?.backgroundColor('#ffcccccc').text('Loading...')
    else instance?.backgroundColor('#ffffffff').text('Loaded')
  }
}
```

## 7. 总结与展望

AttributeStyle的状态管理系统通过标准化状态定义、契约式样式接口和自动化状态流转，解决了传统开发中状态-样式耦合的问题。未来计划引入状态组合（如同时处理Focus+Hover）和状态动画过渡能力，进一步增强交互体验。

## 8. 参考链接
- 项目地址：<mcurl name="AttributeStyle" url="https://gitee.com/qincji/AttributeStyle"></mcurl>
- 官方文档：<mcurl name="自定义修饰器 AttributeModifier" url="https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-user-defined-modifier#attributemodifier"></mcurl>