### HintDialog Advanced Usage and Customization Practices

#### Table of Contents
- [1. Advanced Parameter Configuration Details](#1-advanced-parameter-configuration-details)
- [2. Event Handling and Asynchronous Operations](#2-event-handling-and-asynchronous-operations)
- [3. Custom Styles and Themes](#3-custom-styles-and-themes)
- [4. Complex Content Embedding Techniques](#4-complex-content-embedding-techniques)
- [5. Core Source Code Architecture Analysis](#5-core-source-code-architecture-analysis)
- [6. Comparison with Official Dialog Solution](#6-comparison-with-official-dialog-solution)
- [7. Summary](#7-summary)

#### Abstract
This article explores the advanced features of hint-dialog in depth, including advanced parameter configuration, event handling mechanisms, custom styling solutions, complex content embedding, and source code architecture analysis. It helps developers make full use of this component to implement complex business requirements and compares the advantages and disadvantages with the official solution to provide references for project selection.

#### 1. Advanced Parameter Configuration Details
hint-dialog provides rich parameter configurations to support various complex scenario requirements:

##### 1.1 Complete Parameter List
```typescript
interface HintParm {
  content: string;                // Display content
  title?: string;                 // Title
  noMsg?: string;                 // Cancel button text
  okMsg?: string;                 // Confirm button text
  showNo?: boolean;               // Whether to show cancel button
  isWebType?: boolean;            // Whether it is Web type
  controller?: WebviewController; // Web controller
  outsideCancel?: boolean;        // Whether clicking outside can cancel
  onNo?: () => void;              // Cancel callback
  onOk?: () => void;              // Confirm callback
  alignment?: DialogAlignment;    // Alignment method
}
```

##### 1.2 Web Type Dialog Configuration
When you need to display web content in the dialog, you can configure the isWebType parameter:
```typescript
import { WebviewController } from '@ohos.web.webview';

// Create Web controller
private webController: WebviewController = new WebviewController();

// Open Web type dialog
HintDialog.open({
  title: 'Web Content Display',
  content: 'https://gitee.com/qincji/hint-dialog',
  isWebType: true,
  controller: this.webController,
  showNo: true,
  onOk: () => {
    // Get current URL
    this.webController.getURL().then(url => {
      console.log('Current web address: ' + url);
    });
  }
})
```

#### 2. Event Handling and Asynchronous Operations

##### 2.1 Asynchronous Callback Handling
Dialog buttons support asynchronous callback operations, enabling complex business logic:
```typescript
HintDialog.open({
  title: 'Asynchronous Operation Example',
  content: 'Are you sure you want to submit the form?',
  onOk: async () => {
    // Show loading status
    HintDialog.open({
      content: 'Submitting...',
      showNo: false
    });

    try {
      // Simulate API request
      await new Promise(resolve => setTimeout(resolve, 2000));
      // Close loading dialog
      HintDialog.close();
      // Show success prompt
      HintDialog.open({
        content: 'Submission successful',
        showNo: false
      });
    } catch (e) {
      HintDialog.close();
      HintDialog.open({
        content: 'Submission failed, please try again',
        showNo: false
      });
    }
  }
})
```

##### 2.2 Lifecycle Events
Starting from API version 19, custom popups provide lifecycle functions:
```typescript
HintDialog.initDefaultStyle({
  // Callback before popup appears
  onWillAppear: () => {
    console.log('Dialog is about to show');
  },
  // Callback after popup appears
  onDidAppear: () => {
    console.log('Dialog has shown');
  },
  // Callback before popup disappears
  onWillDisappear: () => {
    console.log('Dialog is about to disappear');
  },
  // Callback after popup disappears
  onDidDisappear: () => {
    console.log('Dialog has disappeared');
  }
})
```
[Official Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-common-components-custom-dialog)

#### 3. Custom Styles and Themes

##### 3.1 Global Style Initialization
You can configure dialog styles globally through the initDefaultStyle method:
```typescript
// Import related classes
import { HintDialog, HintStyle, TitleParm } from '@qincji/dialog';
import { AttributeModifier, TextAttribute, ColumnAttribute } from '@ohos.ui';

// Custom title style
const titleAttr: AttributeModifier<TextAttribute> = {
  applyNormalAttribute(instance: TextAttribute): void {
    instance.fontColor('#ff1145e5')
           .fontWeight(FontWeight.Bolder)
           .fontSize(25)
           .margin({ bottom: 15 });
  }
};

// Custom outer container style
const outBoxAttr: AttributeModifier<ColumnAttribute> = {
  applyNormalAttribute(instance: ColumnAttribute): void {
    instance.backgroundColor('#ffc8f1b3')
           .borderRadius(15)
           .padding(20)
           .margin(50);
  }
};

// Initialize global style
HintDialog.initDefaultStyle({
  titleParm: new TitleParm('Custom Title', titleAttr),
  outBoxAttr: outBoxAttr,
  alignment: DialogAlignment.Bottom,
  outsideCancel: true
})
```

##### 3.2 Button Style Customization
Customize button styles through the btnParm parameter:
```typescript
HintDialog.initDefaultStyle({
  btnParm: {
    okAttr: {
      applyNormalAttribute(btn: ButtonAttribute) {
        btn.backgroundColor('#007dff')
           .fontColor('#ffffff')
           .borderRadius(8)
           .width(120)
           .height(40);
      }
    },
    noAttr: {
      applyNormalAttribute(btn: ButtonAttribute) {
        btn.backgroundColor('#f2f2f2')
           .fontColor('#333333')
           .borderRadius(8)
           .width(120)
           .height(40);
      }
    },
    space: 15 // Button spacing
  }
})
```

#### 4. Complex Content Embedding Techniques

##### 4.1 Embedding Custom Components
The content parameter supports custom component embedding:
```typescript
// Define custom content component
@Component
struct CustomContent {
  @State count: number = 0;

  build() {
    Column({
      space: 10
    }) {
      Text('This is a custom counter')
      Button('+')
        .onClick(() => this.count++)
      Text(`Current value: ${this.count}`)
    }
    .padding(10)
  }
}

// Use in dialog
HintDialog.open({
  title: 'Custom Content Example',
  content: '', // Must be provided, can be empty string
  customContent: () => CustomContent(), // Custom content component
  showNo: true
})
```

##### 4.2 Form Data Collection
Implement form collection using custom content:
```typescript
@State username: string = '';
@State password: string = '';

HintDialog.open({
  title: 'User Login',
  customContent: () => Column({
    space: 15
  }) {
    TextInput({
      placeholder: 'Please enter username'
    })
    .onChange(value => this.username = value)
    .width('100%')

    TextInput({
      placeholder: 'Please enter password',
      type: InputType.Password
    })
    .onChange(value => this.password = value)
    .width('100%')
  }
  .padding(10),
  onOk: () => {
    // Handle login logic
    console.log(`Login information: ${this.username}, ${this.password}`);
  }
})
```

#### 5. Core Source Code Architecture Analysis

hint-dialog's core architecture adopts a singleton pattern design to ensure there is only one dialog instance globally:

```typescript
// Simplified core implementation
class HintDialog {
  private static instance: HintDialog;
  private controller: CustomDialogController;
  private style: HintStyle = defaultStyle;

  // Singleton pattern
  private constructor() {
    this.initController();
  }

  static getInstance(): HintDialog {
    if (!HintDialog.instance) {
      HintDialog.instance = new HintDialog();
    }
    return HintDialog.instance;
  }

  // Initialize dialog controller
  private initController() {
    this.controller = new CustomDialogController({
      builder: this.buildContent,
      style: this.style
    });
  }

  // Open dialog
  static open(params: HintParm) {
    const instance = HintDialog.getInstance();
    instance.updateParams(params);
    instance.controller.open();
  }

  // Other methods...
}
```

The advantages of this design are:
1. Avoids the problem of multiple dialogs popping up simultaneously
2. Unified management of dialog state
3. Simplifies API calls without needing to create instances
4. Facilitates global style configuration

#### 6. Comparison with Official Dialog Solution

| Feature | hint-dialog | Official CustomDialog |
|---------|-------------|-----------------------|
| Usage Complexity | Simple (static method call) | Medium (needs controller creation) |
| Style Customization | Rich API support | Requires manual implementation |
| Feature Completeness | Built-in common functions | Basic functions need extension |
| Code Amount | Less (one-line call) | More (needs component and controller definition) |
| Flexibility | High (supports custom content) | High (fully customizable) |
| Learning Cost | Low | Medium |

Basic usage example of official CustomDialog:
```typescript
// Official CustomDialog implementation
@CustomDialog
struct OfficialDialog {
  controller: CustomDialogController;
  message: string;

  build() {
    Column() {
      Text(this.message)
      Button('Confirm')
        .onClick(() => this.controller.close())
    }
  }
}

// Need to create controller when using
private dialogController: CustomDialogController = new CustomDialogController({
  builder: OfficialDialog({
    message: 'Official dialog',
    controller: this.dialogController
  })
});

// Call to display
this.dialogController.open();
```

#### 7. Summary
This article detailed the advanced usage and customization techniques of hint-dialog, including parameter configuration, event handling, style customization, and complex content embedding. Through source code analysis, we understood its singleton pattern design idea and compared it with the official solution. hint-dialog provides a more concise API while maintaining flexibility, making it suitable for rapid development. The next article will explore best practices and performance optimization strategies of hint-dialog in actual projects.