# AttributeStyle核心技术解析：基于HarmonyOS NEXT的UI样式复用方案

## 1. 引言

在HarmonyOS应用开发过程中，UI样式的重复定义和状态管理一直是影响开发效率的痛点问题。传统方式下，开发者需要为不同组件、不同状态重复编写大量相似的样式代码，不仅增加了维护成本，也导致代码冗余。AttributeStyle项目基于HarmonyOS NEXT 5.0.0的自定义修饰器能力，提出了一种创新的UI样式复用方案，可减少70%的重复代码，显著提升开发效率。

## 2. 官方能力介绍

HarmonyOS NEXT提供了AttributeModifier和AttributeUpdater两种核心能力，支持开发者自定义组件属性和状态更新逻辑：

- **AttributeModifier**：允许开发者自定义组件属性修饰器，实现属性的统一管理和复用<mcurl name="自定义修饰器 AttributeModifier" url="https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-user-defined-modifier#attributemodifier"></mcurl>
- **AttributeUpdater**：提供组件状态变化的回调机制，可根据不同状态动态更新组件样式<mcurl name="自定义扩展属性更新器 AttributeUpdater" url="https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-user-defined-extension-attributeupdater"></mcurl>

## 3. AttributeStyle设计与实现

### 3.1 核心架构

AttributeStyle采用分层设计，主要包含以下核心模块：

- **SAttribute**：继承自AttributeUpdater，作为样式管理的基类
- **SAdapter**：样式适配器，负责管理多个样式的组合与优先级
- **SController**：状态控制器，处理状态变化逻辑并应用相应样式
- **SStyle**：样式接口，定义不同状态下的样式应用规则

### 3.2 关键实现

SAttribute作为核心基类，通过重写AttributeUpdater的回调方法实现状态监听：

```typescript:library/src/main/ets/style/SAttribute.ets
// 系统api 按下的回调
applyPressedAttribute(instance: T): void {
  Log.d(TAG, () => 'applyPressedAttribute...')
  this.controller.stateChange(SState.Pressed, true, instance)
}

// 系统api 按键不可用的回调
applyDisabledAttribute(instance: T): void {
  Log.d(TAG, () => 'applyDisabledAttribute...')
  this.controller.stateChange(SState.Disabled, true, instance)
}
```

SAdapter实现了样式的组合机制，允许将多个样式叠加应用：

```typescript:entry/src/main/ets/pages/Index.ets
private style1 = new SAdapter<TextAttribute>()
  .addStyle(new ContainStyle())
  .addStyle(new BgGreenStyle())
  .addStyle(new TextNormalStyle())
  .addStyle(new TextPressedStyle())
  .addStyle(new HoverBgStyle())
  .buildAttribute(true, true)
```

## 4. 应用效果与优势

### 4.1 代码复用率提升

通过AttributeStyle，一个复杂样式可被多个组件复用。例如在Index页面中，style1定义的组合样式被应用于多个Text组件，避免了重复编写相同的样式代码。

### 4.2 状态管理简化

传统方式需要手动监听并处理各种状态变化，而AttributeStyle通过SController统一管理状态，自动应用对应样式：

```typescript:entry/src/main/ets/pages/Index.ets
Toggle({ type: ToggleType.Switch })
  .onChange((isOn) => {
    this.style5.updateSelect(isOn)
  })
```

## 5. 总结与展望

AttributeStyle基于HarmonyOS NEXT的自定义修饰器能力，通过创新的样式组合和状态管理机制，有效解决了UI开发中的代码冗余问题。未来计划增加更多预设样式库，并优化样式优先级算法，进一步提升开发体验。

## 6. 参考链接
- 项目地址：<mcurl name="AttributeStyle" url="https://gitee.com/qincji/AttributeStyle"></mcurl>
- 官方文档：<mcurl name="自定义扩展修饰器 AttributeModifier" url="https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-user-defined-extension-attributemodifier"></mcurl>