### HintDialog Best Practices and Performance Optimization in Real Projects

#### Table of Contents
- [1. Integration Strategies for Medium to Large Projects](#1-integration-strategies-for-medium-to-large-projects)
- [2. Common Problem Diagnosis and Solutions](#2-common-problem-diagnosis-and-solutions)
- [3. C++/ArkTS Hybrid Programming Performance Optimization](#3-c-arkts-hybrid-programming-performance-optimization)
- [4. Testing Strategies and Automated Test Case Design](#4-testing-strategies-and-automated-test-case-design)
- [5. Community Feedback and Future Plans](#5-community-feedback-and-future-plans)
- [6. Practical Code Snippets and Resource Links](#6-practical-code-snippets-and-resource-links)
- [7. Summary](#7-summary)

#### Abstract
This article shares integration experience of hint-dialog in medium to large projects, deeply analyzes solutions to common issues such as memory leaks and duplicate pop-ups, explores how to improve performance through ArkTS hybrid programming, and provides testing strategies and automated test case design suggestions to help developers build efficient and stable dialog systems.

#### 1. Integration Strategies for Medium to Large Projects

##### 1.1 Global Configuration and Encapsulation
In medium to large projects, it is recommended to secondary encapsulate hint-dialog to uniformly manage dialog styles and behaviors:
```typescript
// utils/DialogManager.ets
import { HintDialog, HintStyle, TitleParm } from '@qincji/dialog';

export class DialogManager {
  // Initialize global style
  static initGlobalStyle() {
    HintDialog.initDefaultStyle({
      titleParm: new TitleParm('System Prompt'),
      outsideCancel: false, // It is recommended to disable outside cancel by default in production environment
      alignment: DialogAlignment.Center,
      // Unified button style
      btnParm: {
        okAttr: { /* Production environment button style */ },
        noAttr: { /* Production environment button style */ }
      }
    });
  }

  // Encapsulate common dialog types
  static showSuccess(message: string) {
    return HintDialog.open({
      content: message,
      showNo: false,
      // Success style customization
    });
  }

  static showError(message: string) {
    return HintDialog.open({
      content: message,
      showNo: false,
      // Error style customization
    });
  }

  static showConfirm(message: string, onOk: () => void) {
    return HintDialog.open({
      content: message,
      onOk: onOk
    });
  }
}
```

Initialize at the application entry:
```typescript
// main/AbilityStage.ts
import { DialogManager } from '../utils/DialogManager';

export default class MyAbilityStage extends AbilityStage {
  onCreate() {
    super.onCreate();
    DialogManager.initGlobalStyle(); // Initialize dialog global style
  }
}
```

##### 1.2 Modular Usage Strategy
Divide dialog configurations according to business modules to avoid style conflicts:
```typescript
// Business module-specific dialog configuration
export namespace OrderDialog {
  export function showPaymentConfirm(amount: number, onConfirm: () => void) {
    HintDialog.open({
      title: 'Order Payment',
      content: `Confirm payment of ${amount} yuan?`,
      okMsg: 'Pay Now',
      onOk: onConfirm,
      // Order module-specific style
      outBoxAttr: {/* Order module style */}
    });
  }
}
```

#### 2. Common Problem Diagnosis and Solutions

##### 2.1 Memory Leak Issues
**Problem表现**: Memory usage continues to increase after frequently opening and closing dialogs
**原因分析**: Unreleased resource references exist in callback functions
**解决方案**: Use weak references and lifecycle management
```typescript
// Wrong example: May cause memory leaks
HintDialog.open({
  onOk: () => {
    this.dataService.fetchData(); // This reference prevents dialog instance from being released
  }
});

// Correct example: Use weak reference
const weakThis = new WeakRef(this);
HintDialog.open({
  onOk: () => {
    const that = weakThis.get();
    if (that) {
      that.dataService.fetchData();
    }
  }
});
```

##### 2.2 Duplicate Pop-up Issues
**Problem表现**: Multiple dialogs pop up叠加 when clicking buttons quickly
**解决方案**: Add pop-up status lock
```typescript
// Add status control in DialogManager
private static isShowing = false;

static showConfirm(message: string, onOk: () => void) {
  if (DialogManager.isShowing) return;
  DialogManager.isShowing = true;

  return HintDialog.open({
    content: message,
    onOk: () => {
      onOk();
      DialogManager.isShowing = false;
    },
    onNo: () => {
      DialogManager.isShowing = false;
    },
    onDidDisappear: () => {
      DialogManager.isShowing = false; // Ensure status is reset in all cases
    }
  });
}
```
[Official Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-common-components-custom-dialog)

##### 2.3 Animation Stutter Optimization
**Problem表现**: Dialog pop-up/disappearance animation stutters
**Optimization方案**:
1. Reduce dialog content complexity
2. Avoid performing heavy operations during animation
3. Use hardware accelerated rendering
```typescript
// Optimize animation performance
HintDialog.initDefaultStyle({
  // Simplify animation configuration
  openAnimation: { duration: 200, curve: Curve.EaseOut },
  closeAnimation: { duration: 150, curve: Curve.EaseIn },
  outBoxAttr: {
    applyNormalAttribute: (column) => {
      column
        .backgroundColor('#ffffff')
        .borderRadius(12)
        .clip(true) // Enable clipping optimization
        .transition(TransitionEffect.OPACITY | TransitionEffect.SCALE);
    }
  }
});
```

##### 3.1 Rendering Optimization
Use ArkTS to implement complex graphics drawing and display through ArkUI's custom drawing components:
```typescript
// Custom drawing component with C++ backend
@Component
struct ChartView {
  @Prop data: number[];

  build() {
    CustomPaint({
      painter: new ChartPainter(this.data), // C++ implemented painter
      size: { width: '100%', height: 200 }
    })
  }
}

// Use high-performance chart in dialog
HintDialog.open({
  title: 'Data Statistics',
  customContent: () => ChartView({
    data: [12, 34, 23, 56, 45]
  })
});
```

#### 4. Testing Strategies and Automated Test Case Design

##### 4.1 Unit Testing
Use HarmonyOS testing framework to perform unit testing on dialog logic:
```typescript
// test/unit/DialogManager.test.ets
import { DialogManager } from '../../utils/DialogManager';
import { HintDialog } from '@qincji/dialog';

describe('DialogManager', () => {
  beforeAll(() => {
    DialogManager.initGlobalStyle();
  });

  test('showSuccess should open dialog with correct parameters', () => {
    // Mock HintDialog.open
    const mockOpen = spyOn(HintDialog, 'open');
    DialogManager.showSuccess('Test success');
    expect(mockOpen).toHaveBeenCalledWith(expect.objectContaining({
      content: 'Test success',
      showNo: false
    }));
  });
});
```

##### 4.2 UI Automation Testing
Use UI testing framework to verify dialog interactions:
```typescript
// test/ui/DialogUITest.ets
import { Driver, By, until } from '@ohos.uiautomator';

describe('HintDialog UI Test', () => {
  let driver: Driver;

  beforeAll(async () => {
    driver = await Driver.create();
    await driver.launchApp('com.example.myapp');
  });

  test('dialog should respond to ok click', async () => {
    // Click button to show dialog
    await driver.findElement(By.text('Show Dialog')).click();
    // Verify dialog display
    const dialog = await driver.findElement(By.text('System Prompt'));
    expect(await dialog.isDisplayed()).toBe(true);
    // Click confirm button
    await driver.findElement(By.text('Confirm')).click();
    // Verify dialog closed
    expect(await driver.findElement(By.text('System Prompt')).isDisplayed()).toBe(false);
  });
});
```

#### 5. Community Feedback and Future Plans

##### 5.1 Community Contribution Guidelines
hint-dialog welcomes community contributions. The contribution process is as follows:
1. Fork the project to your personal repository
2. Create a feature branch: `git checkout -b feature/xxx`
3. Commit code: `git commit -m 'feat: add xxx feature'`
4. Push branch: `git push origin feature/xxx`
5. Create Pull Request

Contributors should follow code specifications, ensure test coverage, and describe feature purposes and implementation ideas in the PR.

##### 5.2 Future Feature Plans
Based on community feedback, hint-dialog plans to support in future versions:
- Dialog queue management, supporting sequential display of multiple dialogs
- Custom animation effect library, providing more transition animation options
- Accessibility support, optimizing screen reader compatibility
- Multi-language internationalization support, with built-in common language packs
- Theme switching function, supporting automatic adaptation of light/dark modes

#### 6. Practical Code Snippets and Resource Links

**Complete implementation of common utility class**:
```typescript
// Complete dialog utility class example
import { HintDialog, HintParm } from '@qincji/dialog';
import { Log } from '../utils/Log';

export class DialogUtil {
  private static dialogStack: number = 0;
  private static readonly MAX_STACK = 3;

  static open(params: HintParm) {
    if (this.dialogStack >= this.MAX_STACK) {
      Log.warn('Dialog stack overflow, ignore new dialog');
      return;
    }

    this.dialogStack++;
    const originalOnOk = params.onOk;
    const originalOnNo = params.onNo;
    const originalDisappear = params.onDidDisappear;

    // Wrap callback functions to manage dialog stack
    params.onOk = () => {
      originalOnOk?.();
      this.dialogStack--;
    };

    params.onNo = () => {
      originalOnNo?.();
      this.dialogStack--;
    };

    params.onDidDisappear = () => {
      originalDisappear?.();
      this.dialogStack--;
    };

    return HintDialog.open(params);
  }

  static closeAll() {
    HintDialog.close();
    this.dialogStack = 0;
  }
}
```

**External resource links**:
- [hint-dialog Issue Tracking](https://gitee.com/qincji/hint-dialog/issues)
- [HarmonyOS Performance Optimization Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/performance-optimization-overview)
- [HarmonyOS Testing Framework](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/test-introduction)

#### 7. Summary
This article systematically introduces the integration strategies of hint-dialog in medium to large projects, deeply analyzes solutions to common problems such as memory leaks and duplicate pop-ups, and explores methods to improve performance through ArkTS programming. At the same time, it provides comprehensive testing strategies, automated test case design suggestions, as well as community contribution guidelines and future plans. Through these best practices, developers can give full play to the advantages of hint-dialog and build an efficient, stable, and user-friendly dialog system. hint-dialog will continue to iterate and optimize, and developers are welcome to participate in contributions and feedback.