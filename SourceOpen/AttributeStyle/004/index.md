# AttributeStyle Custom Adapter Development Guide: From Theory to Practice

## 1. Introduction

Adapters are the core载体 for AttributeStyle to实现 style reuse and business customization. They decouple general style logic from specific business requirements, allowing developers to create highly customized style solutions for different component types. This article systematically introduces the design ideas, implementation methods, and typical application scenarios of custom adapters to help developers quickly master this key technology and build style systems that meet business needs.

## 2. Adapter Design Philosophy

The adapter pattern of AttributeStyle is based on the following design principles:
- **Single Responsibility**: Each adapter focuses on one type of component or one business scenario
- **Open/Closed Principle**: Extend new styles through inheritance rather than modifying existing code
- **Composition Over Inheritance**: Support combining multiple basic styles into complex styles
- **Convention Over Configuration**: Follow unified state method naming conventions

This design makes style code highly maintainable and extensible, especially suitable for UI style management in large-scale projects.

## 3. Basic Implementation of Custom Adapters

### 3.1 Adapter Development Steps

Developing a custom adapter requires completing the following steps:
1. Inherit the `SAdapter` base class and specify the target component attribute type
2. Override the state style methods that need customization
3. Apply through `attributeModifier` in the page

### 3.2 Basic Adapter Example: CustomTextAdapter

CustomTextAdapter provides unified styles for text components and implements style definitions for multiple states:

```typescript:entry/src/main/ets/demo/adapter/CustomTextAdapter.ets
export class CustomTextAdapter extends SAdapter<TextAttribute> {
  // Initialization style (executed only once)
  initializeModifier(instance?: TextAttribute): void {
    instance?.fontSize(12)
      .textAlign(TextAlign.Center)
      .backgroundColor(Color.Transparent)
      .borderRadius(3)
      .padding({ left: 15, right: 15, top: 5, bottom: 5 })
  }

  // Normal state style
  normalStyle(instance?: TextAttribute): void {
    instance?.fontColor(Color.Black).backgroundColor('#22000000').borderWidth(0)
  }

  // Pressed state style
  pressedStyle(instance?: TextAttribute): void {
    instance?.fontColor(Color.White).backgroundColor('#66000000')
  }

  // Other state implementations...
}
```

**Key Technical Points**:
- `initializeModifier`: Sets fixed attributes (such as padding, rounded corners), executed only during component initialization
- State methods (normalStyle, pressedStyle, etc.): Dynamically modify attributes according to state
- Chain calls: Use ArkTS's chain API to simplify attribute settings

## 4. Specialized Adapter Development Cases

### 4.1 Input Box Adapter: CustomInputAdapter

Targeting the special needs of text input boxes, CustomInputAdapter customizes input box-specific attributes such as cursor style and placeholder style:

```typescript:entry/src/main/ets/demo/adapter/CustomInputAdapter.ets
export class CustomInputAdapter extends SAdapter<TextInputAttribute> {
  normalStyle(instance?: TextInputAttribute): void {
    instance?.caretColor('#ff62fc66') // Cursor color
      .caretStyle({ width: 1 }) // Cursor width
      .placeholderFont({ size: 12 }) // Placeholder font
      .placeholderColor('#66000000') // Placeholder color
      .focusable(true)
      .focusOnTouch(true)
  }

  focusStyle(instance?: TextInputAttribute, appear?: boolean): void {
    instance?.borderColor('#fffc1c84') // Focus border color
  }

  // Other state implementations...
}
```

**Input Box Adaptation Points**:
- Cursor style customization (color, width)
- Placeholder style settings
- Focus behavior control (focusable, focusOnTouch)
- Input text style unification

### 4.2 Combined Style Adapter: PacketAdapter

PacketAdapter directly combines multiple basic styles through the constructor to实现 rapid style reuse:

```typescript:entry/src/main/ets/demo/adapter/PacketAdapter.ets
export class PacketAdapter<T extends CommonAttribute> extends SAdapter<T> {
  constructor() {
    super([
      new ContainStyle(),        // Layout style
      new ContainNormalStyle(),  // Container normal style
      new PressedBgStyle(),      // Pressed background style
      new HoverBgStyle()         // Hover background style
    ])
  }
}
```

**Combination Advantages**:
- Code reuse: Basic styles can be shared among multiple adapters
- Flexible configuration: Achieve different effects by adjusting style order and combinations
- Low maintenance cost: Basic style modifications are automatically applied to all combined adapters

## 5. Application of Adapters in Pages

### 5.1 Basic Application Method

Create an adapter instance in the page and apply it to the component through `attributeModifier`:

```typescript:entry/src/main/ets/pages/Index.ets
// Create adapter instances
private style2 = new CustomTextAdapter().buildAttribute()
private style8 = new CustomInputAdapter().buildAttribute()

// Apply to components
Text('style2 CustomTextAdapter')
  .attributeModifier(this.style2)

TextInput({ placeholder: "style8 focus" })
  .attributeModifier(this.style8)
```

### 5.2 Dynamic State Control

Dynamically update component state through methods provided by the adapter:

```typescript:entry/src/main/ets/pages/Index.ets
// Toggle selected state
Toggle({ type: ToggleType.Switch })
  .onChange((isOn) => {
    this.style5.updateSelect(isOn)
  })
```

## 6. Adapter Development Best Practices

### 6.1 Code Organization Recommendations
- Classify adapters by component type (such as text, input, button)
- Separate basic styles and business styles for storage
- Use TypeScript generics to support multiple component types

### 6.2 Performance Optimization Techniques
- Avoid creating new objects in state methods
- Cache complex calculation results
- Reduce unnecessary attribute settings (only modify changed attributes)

### 6.3 Extensibility Design
- Reserve custom configuration interfaces:
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

## 7. Summary and Outlook

Custom adapters are the core embodiment of AttributeStyle's flexibility. Through the methods introduced in this article, developers can build style systems that meet various business needs. In the future, AttributeStyle plans to provide more preset adapter libraries and visual configuration tools to further reduce the threshold for style development.

## 8. Reference Links
- Project Address: <mcurl name="AttributeStyle" url="https://gitee.com/qincji/AttributeStyle"></mcurl>
- Official Documentation: <mcurl name="自定义扩展修饰器 AttributeModifier" url="https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-user-defined-extension-attributemodifier"></mcurl>