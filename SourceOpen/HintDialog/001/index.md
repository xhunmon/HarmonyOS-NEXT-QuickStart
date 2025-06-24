### HintDialog Basic Introduction and Quick Start

#### Table of Contents
- [1. Background and Application Scenarios of HarmonyOS Custom Dialogs](#1-background-and-application-scenarios-of-harmonyos-custom-dialogs)
- [2. Design Philosophy and Advantages of hint-dialog](#2-design-philosophy-and-advantages-of-hint-dialog)
- [3. Project Address and Installation Method](#3-project-address-and-installation-method)
- [4. Quick Integration and Basic Usage](#4-quick-integration-and-basic-usage)
- [5. Hello World Example and Parameter Description](#5-hello-world-example-and-parameter-description)
- [6. API Documentation and Resource Links](#6-api-documentation-and-resource-links)
- [7. Summary](#7-summary)

#### Abstract
This article will introduce the basic usage of the custom dialog component hint-dialog under the HarmonyOS platform, including background introduction, installation steps, quick integration examples, and core API explanations, helping developers quickly get started with this powerful dialog component.

#### 1. Background and Application Scenarios of HarmonyOS Custom Dialogs
In mobile application development, dialogs are important components for interacting with users, widely used in scenarios such as information prompts, operation confirmation, and input collection. As a new generation of intelligent terminal operating system, HarmonyOS provides rich UI components, among which CustomDialog allows developers to customize pop-up content and interaction logic according to business needs [Official Documentation](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-common-components-custom-dialog).

However, the native CustomDialog has problems such as cumbersome usage steps, complex style customization, and single function in actual development. To solve these pain points, the hint-dialog open source library came into being, aiming to provide a set of easy-to-use and highly customizable dialog solutions.

#### 2. Design Philosophy and Advantages of hint-dialog
The design philosophy of hint-dialog is "Simple API, Rich Functions", with main advantages including:

- **Easy to use**: Direct call through static methods without complex initialization process
- **Flexible styling**: Supports comprehensive style customization of title, content, buttons, etc.
- **Comprehensive functionality**: Supports text/web content display, multi-button configuration, event callbacks, etc.
- **Good compatibility**: Adapts to different versions of HarmonyOS and provides a consistent user experience
- **Lightweight and efficient**: Streamlined core code with optimized performance

Compared with the official CustomDialog, hint-dialog encapsulates more commonly used functions, reducing repetitive coding work and allowing developers to focus on business logic implementation rather than UI details [Project Repository](https://gitee.com/qincji/hint-dialog).

#### 3. Project Address and Installation Method
**Project Address**: [hint-dialog Open Source Repository](https://gitee.com/qincji/hint-dialog)

**Installation Method**:
Use ohpm (OpenHarmony Package Manager) for installation:
```bash
ohpm install @qincji/dialog
```

> Note: Please ensure that your development environment has ohpm configured and supports HarmonyOS API Version 9 or higher.

#### 4. Quick Integration and Basic Usage
Integrating hint-dialog into your project requires just the following steps:

##### 4.1 Import Component
Import HintDialog in the page where you need to use the dialog:
```typescript
import { HintDialog } from '@qincji/dialog';
```

##### 4.2 Initialize Component
Add HintDialog initialization code in the build method of the application's main page (usually MainPage.ets):
```typescript
@Entry
@Component
struct MainPage {
  build() {
    Row() {
      // Other page content
      HintDialog() // Initialize HintDialog
    }
    .width('100%')
    .height('100%')
  }
}
```

> Note: HintDialog only needs to be initialized once on the main page and is available globally. There is no need to reinitialize after page routing jumps [Project Details](https://gitee.com/qincji/hint-dialog).

#### 5. Hello World Example and Parameter Description
The following is a simplest example of using HintDialog:

```typescript
// Call in button click event
Button('Show Prompt')
  .onClick(() => {
    HintDialog.open({
      content: 'This is my first HintDialog',
      showNo: false,
      onOk: () => {
        console.log('User clicked OK button');
      }
    })
  })
```

##### Parameter Description:
- **content**: The content displayed in the dialog, which is text or web content depending on the isWebType property
- **title**: Title text, default: "Warm Prompt"
- **showNo**: Whether to display the cancel button, default: true
- **okMsg**: OK button text, default: "OK"
- **noMsg**: Cancel button text, default: "Cancel"
- **onOk**: OK button click callback function
- **onNo**: Cancel button click callback function
- **isWebType**: Whether it is web type content, default: false
- **outsideCancel**: Whether clicking the outside area can cancel, default: true
- **alignment**: Dialog position, default: center

#### 6. API Documentation and Resource Links
- [hint-dialog Complete API Documentation](https://gitee.com/qincji/hint-dialog/blob/master/README.md)
- [HarmonyOS Official CustomDialog Documentation](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-common-components-custom-dialog)
- [OpenHarmony Application Development Guide](https://developer.huawei.com/consumer/cn/doc/development/arkUI-ts/arkts-guides)

#### 7. Summary
This article introduces the basic usage of hint-dialog, including installation steps, initialization process, and simple examples. With hint-dialog, developers can quickly implement fully functional dialogs, greatly reducing development workload. The next article will delve into advanced usage and customization techniques of hint-dialog, helping developers build dialog components that better meet business needs.