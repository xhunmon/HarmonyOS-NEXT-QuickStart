# AttributeStyle样式组合机制：构建灵活可复用的UI样式系统

## 1. 引言

在复杂UI开发中，单一组件往往需要同时应用多种样式（如布局、颜色、交互状态等），传统开发方式中样式代码的碎片化管理导致维护困难。AttributeStyle通过创新的样式组合机制，允许开发者将多个独立样式单元灵活组合，实现样式的模块化复用与精确控制，进一步提升代码复用率和开发效率。

## 2. 官方能力基础

HarmonyOS NEXT的自定义扩展修饰器（AttributeModifier）支持开发者为组件定义统一的属性修改逻辑，但原生能力未提供样式的组合与优先级管理机制<mcurl name="自定义扩展修饰器 AttributeModifier" url="https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-user-defined-extension-attributemodifier"></mcurl>。AttributeStyle基于此能力构建了多层级的样式组合架构，填补了这一空白。

## 3. 样式组合核心实现

### 3.1 SAdapter：样式组合的核心引擎

SAdapter作为样式组合的适配器，提供了`addStyle`方法实现多样式单元的有序叠加，并通过`buildAttribute`方法生成可应用于组件的属性修饰器：

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

**关键特性**：
- 支持链式调用，简化多样式组合代码
- 维护样式执行顺序，后添加的样式可覆盖先添加样式的同名属性
- 与SAttribute无缝集成，将组合样式转换为组件可识别的修饰器

### 3.2 多样式协同执行机制

SAdapter通过`notifyAll`方法实现对所有组合样式的统一调度，确保状态变化时所有相关样式同步更新：

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

以`pressedStyle`为例，当组件触发按压状态时，所有组合样式的按压逻辑将依次执行：

```typescript:library/src/main/ets/style/SAdapter.ets
pressedStyle(instance?: T | undefined): void {
  Log.d(TAG, () => 'pressedStyle...')
  this.notifyAll((style) => {
    style.pressedStyle?.(instance)
  })
}
```

## 4. 状态驱动的样式调度：SController

SController作为状态控制器，负责将组件状态变化映射为样式执行指令，其核心逻辑如下：

```typescript:library/src/main/ets/style/SController.ets
stateChange(state: SState, isAppear: boolean, instance?: T | undefined) {
  if (!isAppear && this.disappearToNormalStyle) { // 状态消失时恢复正常样式
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
    // 其他状态处理...
  }
}
```

**工作流程**：
1. 接收状态变化事件（如按压、悬停、选中）
2. 根据状态类型调用SAdapter中对应的样式方法
3. 状态消失时可自动恢复正常样式（通过disappearToNormalStyle配置）

## 5. 实战应用案例

在Index页面中，通过SAdapter组合5种独立样式，构建复杂按钮效果：

```typescript:entry/src/main/ets/pages/Index.ets
private style1 = new SAdapter<TextAttribute>()
  .addStyle(new ContainStyle()) // 布局样式
  .addStyle(new BgGreenStyle()) // 背景样式
  .addStyle(new TextNormalStyle()) // 文本默认样式
  .addStyle(new TextPressedStyle()) // 文本按压样式
  .addStyle(new HoverBgStyle()) // 悬停背景样式
  .buildAttribute(true, true)
```

**优势分析**：
- **模块化**：每种样式独立维护，如ContainStyle专注布局控制
- **可复用**：相同样式组合可在多个组件中共享
- **可扩展**：新增样式只需实现SStyle接口并通过addStyle添加

## 6. 总结与展望

AttributeStyle的样式组合机制通过SAdapter和SController的协同工作，解决了传统样式管理中的代码碎片化问题。未来计划引入样式优先级权重系统和条件样式规则，进一步增强样式组合的灵活性。

## 7. 参考链接
- 项目地址：<mcurl name="AttributeStyle" url="https://gitee.com/qincji/AttributeStyle"></mcurl>
- 官方文档：<mcurl name="自定义扩展属性更新器 AttributeUpdater" url="https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-user-defined-extension-attributeupdater"></mcurl>