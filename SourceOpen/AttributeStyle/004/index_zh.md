# AttributeStyle自定义适配器开发指南：从理论到实践

## 1. 引言

适配器(Adapter)是AttributeStyle实现样式复用与业务定制的核心载体，它将通用样式逻辑与特定业务需求解耦，允许开发者为不同组件类型创建高度定制化的样式方案。本文将系统介绍自定义适配器的设计思想、实现方法及典型应用场景，帮助开发者快速掌握这一关键技术，构建符合业务需求的样式系统。

## 2. 适配器设计理念

AttributeStyle的适配器模式基于以下设计原则：
- **单一职责**：每个适配器专注于一类组件或一种业务场景
- **开闭原则**：通过继承扩展新样式，而非修改现有代码
- **组合优于继承**：支持将多个基础样式组合为复杂样式
- **约定优于配置**：遵循统一的状态方法命名规范

这种设计使样式代码具备高度可维护性和可扩展性，特别适合大型项目的UI样式管理。

## 3. 自定义适配器基础实现

### 3.1 适配器开发步骤

开发自定义适配器需完成以下步骤：
1. 继承`SAdapter`基类并指定目标组件属性类型
2. 重写需要定制的状态样式方法
3. 在页面中通过`attributeModifier`应用

### 3.2 基础适配器示例：CustomTextAdapter

CustomTextAdapter为文本组件提供统一样式，实现了多种状态的样式定义：

```typescript:entry/src/main/ets/demo/adapter/CustomTextAdapter.ets
export class CustomTextAdapter extends SAdapter<TextAttribute> {
  // 初始化样式（仅执行一次）
  initializeModifier(instance?: TextAttribute): void {
    instance?.fontSize(12)
      .textAlign(TextAlign.Center)
      .backgroundColor(Color.Transparent)
      .borderRadius(3)
      .padding({ left: 15, right: 15, top: 5, bottom: 5 })
  }

  // 正常状态样式
  normalStyle(instance?: TextAttribute): void {
    instance?.fontColor(Color.Black).backgroundColor('#22000000').borderWidth(0)
  }

  // 按压状态样式
  pressedStyle(instance?: TextAttribute): void {
    instance?.fontColor(Color.White).backgroundColor('#66000000')
  }

  // 其他状态实现...
}
```

**关键技术点**：
- `initializeModifier`：设置固定属性（如内边距、圆角），仅在组件初始化时执行
- 状态方法（normalStyle、pressedStyle等）：根据状态动态修改属性
- 链式调用：使用ArkTS的链式API简化属性设置

## 4. 专项适配器开发案例

### 4.1 输入框适配器：CustomInputAdapter

针对文本输入框的特殊需求，CustomInputAdapter定制了光标样式、占位符样式等输入框特有属性：

```typescript:entry/src/main/ets/demo/adapter/CustomInputAdapter.ets
export class CustomInputAdapter extends SAdapter<TextInputAttribute> {
  normalStyle(instance?: TextInputAttribute): void {
    instance?.caretColor('#ff62fc66') // 光标颜色
      .caretStyle({ width: 1 }) // 光标宽度
      .placeholderFont({ size: 12 }) // 占位符字体
      .placeholderColor('#66000000') // 占位符颜色
      .focusable(true)
      .focusOnTouch(true)
  }

  focusStyle(instance?: TextInputAttribute, appear?: boolean): void {
    instance?.borderColor('#fffc1c84') // 获焦边框颜色
  }

  // 其他状态实现...
}
```

**输入框适配要点**：
- 光标样式定制（颜色、宽度）
- 占位符样式设置
- 焦点行为控制（focusable、focusOnTouch）
- 输入文本样式统一

### 4.2 组合样式适配器：PacketAdapter

PacketAdapter通过构造函数直接组合多个基础样式，实现样式的快速复用：

```typescript:entry/src/main/ets/demo/adapter/PacketAdapter.ets
export class PacketAdapter<T extends CommonAttribute> extends SAdapter<T> {
  constructor() {
    super([
      new ContainStyle(),        // 布局样式
      new ContainNormalStyle(),  // 容器正常样式
      new PressedBgStyle(),      // 按压背景样式
      new HoverBgStyle()         // 悬停背景样式
    ])
  }
}
```

**组合优势**：
- 代码复用：基础样式可在多个适配器中共享
- 配置灵活：通过调整样式顺序和组合实现不同效果
- 维护成本低：基础样式修改自动应用到所有组合适配器

## 5. 适配器在页面中的应用

### 5.1 基础应用方式

在页面中创建适配器实例并通过`attributeModifier`应用到组件：

```typescript:entry/src/main/ets/pages/Index.ets
// 创建适配器实例
private style2 = new CustomTextAdapter().buildAttribute()
private style8 = new CustomInputAdapter().buildAttribute()

// 应用到组件
Text('style2 CustomTextAdapter')
  .attributeModifier(this.style2)

TextInput({ placeholder: "style8 focus" })
  .attributeModifier(this.style8)
```

### 5.2 动态状态控制

通过适配器提供的方法动态更新组件状态：

```typescript:entry/src/main/ets/pages/Index.ets
// 切换选中状态
Toggle({ type: ToggleType.Switch })
  .onChange((isOn) => {
    this.style5.updateSelect(isOn)
  })
```

## 6. 适配器开发最佳实践

### 6.1 代码组织建议
- 将适配器按组件类型分类（如text、input、button）
- 基础样式与业务样式分离存储
- 使用TypeScript泛型支持多种组件类型

### 6.2 性能优化技巧
- 避免在状态方法中创建新对象
- 复杂计算结果缓存
- 减少不必要的属性设置（仅修改变化的属性）

### 6.3 扩展性设计
- 预留自定义配置接口：
```typescript
class ConfigurableAdapter extends SAdapter<TextAttribute> {
  constructor(private config: { fontSize: number, color: string }) {
    super()
  }

  normalStyle(instance?: TextAttribute): void {
    instance?.fontSize(this.config.fontSize).fontColor(this.config.color)
  }
}
```

## 7. 总结与展望

自定义适配器是AttributeStyle灵活性的核心体现，通过本文介绍的方法，开发者可以构建满足各种业务需求的样式系统。未来AttributeStyle计划提供更多预设适配器库和可视化配置工具，进一步降低样式开发门槛。

## 8. 参考链接
- 项目地址：<mcurl name="AttributeStyle" url="https://gitee.com/qincji/AttributeStyle"></mcurl>
- 官方文档：<mcurl name="自定义扩展修饰器 AttributeModifier" url="https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-user-defined-extension-attributemodifier"></mcurl>