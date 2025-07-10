# Nearly 200 Development Cases

## Introduction
In the process of HarmonyOS application development, referring to practical cases is an important way to improve development efficiency and solve problems. Huawei officially provides a project containing nearly 200 development cases, covering various aspects such as common component implementation, function development, and performance optimization, providing developers with valuable practical references.

## Official Resources Introduction
This case project is hosted on the Gitee platform, with the official address: https://developer.huawei.com/consumer/cn/forum/topic/0208141669934007200?fid=0109140870620153026. The project contains rich case resources covering implementations from basic components to complex functions, suitable for developers at different levels to learn and reference.

## Detailed Explanation
### Case Project Overview
The HarmonyOS case project is a comprehensive open-source project that brings together various practical cases contributed by Huawei officials and community developers. These cases are developed based on different versions of HarmonyOS, involving multiple development scenarios such as UI design, state management, network requests, and multi-device adaptation.

### Case Content Classification
1. **Basic Component Cases**: Include usage examples of common UI components such as buttons, lists, and forms, helping developers quickly master basic component usage.
2. **Function Implementation Cases**: Cover implementation solutions for common functions such as page routing, data storage, audio and video playback, and map integration.
3. **Performance Optimization Cases**: Provide practical experience in startup speed optimization, memory management, rendering performance improvement, etc.
4. **Industry Application Cases**: Include application examples in education, finance, medical and other industries, demonstrating the application potential of HarmonyOS in different fields.

### How to Effectively Use Cases
1. **Search on Demand**: According to development needs, locate relevant cases through project directory structure or search function.
2. **Source Code Analysis**: Carefully read case source code to understand implementation ideas and key technical points.
3. **Local Running**: Import cases into DevEco Studio, run and debug to observe actual effects.
4. **Secondary Development**: Modify and extend based on cases to adapt to own project requirements.

## Practical Suggestions
1. **Combine with Documentation Learning**: Read cases together with official documentation to deeply understand technical principles.
2. **Pay Attention to Version Adaptation**: Note the HarmonyOS version corresponding to the case and select cases matching the project version for reference.
3. **Participate in Community Communication**: Communicate experiences with other developers in the case project's Issue or discussion area.
4. **Regularly Update Case Library**: Pay attention to project updates to obtain the latest case resources and best practices.
5. **Summarize Case Patterns**: Induce implementation patterns of different cases to form own development methodology.

## Summary and Outlook
Nearly 200 development cases provide rich practical resources for HarmonyOS developers, effectively lowering the development threshold. With the continuous development of the HarmonyOS ecosystem, the case library will continue to expand and improve. Developers should make full use of these cases, combine with their own project needs, and improve development capabilities and product quality.

## Case - Text Selection Menu

The following is the core implementation code of the text selection menu, extending the custom selection menu function through the rich text component:

```typescript
import { RichEditor, RichEditorController } from '@ohos.component.richEditor';
import promptAction from '@ohos.promptAction';

@Component
struct RichEditorComponent {
  private controller: RichEditorController = new RichEditorController();
  private menuItemsContent = [
    { value: '翻译', action: () => this.handleMenuClick('翻译') },
    { value: '搜索', action: () => this.handleMenuClick('搜索') },
    { value: '分享', action: () => this.handleMenuClick('分享') }
  ];

  // Custom menu creation
  private onCreateMenu = (textMenuItems: Array<TextMenuItem>) => {
    this.menuItemsContent.forEach(item => {
      textMenuItems.push({
        value: item.value,
        action: item.action
      });
    });
    return textMenuItems;
  };

  // Menu click event handling
  private onMenuItemClick = (item: TextMenuItem, textRange: TextRange) => {
    promptAction.showToast({ message: `执行${item.value}操作: ${textRange.text}` });
  };

  build() {
    Column() {
      RichEditor({
        controller: this.controller
      })
      .width('100%')
      .height(300)
      .placeholder('请输入文本...')
      // Configure custom selection menu
      .editMenuOptions({
        onCreateMenu: this.onCreateMenu,
        onMenuItemClick: this.onMenuItemClick
      })
    }
    .padding(16)
  }
}
```

### Code Description
1. **Complete Content**: https://developer.huawei.com/consumer/cn/forum/topic/0207166204736188374?fid=0109140870620153026
2. **Rich Text Controller**: Use `RichEditorController` to manage editing status and support text selection range acquisition
3. **Interactive Feedback**: Combine `promptAction` to implement Toast prompts for operation results