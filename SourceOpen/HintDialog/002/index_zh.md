### HintDialog 高级用法与定制化实践

#### 目录
- [1. 高级参数配置详解](#1-高级参数配置详解)
- [2. 事件处理与异步操作](#2-事件处理与异步操作)
- [3. 自定义样式与主题](#3-自定义样式与主题)
- [4. 复杂内容嵌入技巧](#4-复杂内容嵌入技巧)
- [5. 核心源码架构分析](#5-核心源码架构分析)
- [6. 与官方Dialog方案对比](#6-与官方Dialog方案对比)
- [7. 小结](#7-小结)

#### 摘要
本文深入探讨hint-dialog的高级特性，包括高级参数配置、事件处理机制、自定义样式方案、复杂内容嵌入以及源码架构分析，帮助开发者充分利用该组件实现复杂业务需求，并对比官方方案的优缺点，为项目选型提供参考。

#### 1. 高级参数配置详解
hint-dialog提供了丰富的参数配置，支持各种复杂场景需求：

##### 1.1 完整参数列表
```typescript
interface HintParm {
  content: string;                // 显示内容
  title?: string;                 // 标题
  noMsg?: string;                 // 取消按钮文本
  okMsg?: string;                 // 确定按钮文本
  showNo?: boolean;               // 是否显示取消按钮
  isWebType?: boolean;            // 是否为Web类型
  controller?: WebviewController; // Web控制器
  outsideCancel?: boolean;        // 点击外部是否可取消
  onNo?: () => void;              // 取消回调
  onOk?: () => void;              // 确定回调
  alignment?: DialogAlignment;    // 对齐方式
}
```

##### 1.2 Web类型对话框配置
当需要在对话框中展示网页内容时，可配置isWebType参数：
```typescript
import { WebviewController } from '@ohos.web.webview';

// 创建Web控制器
private webController: WebviewController = new WebviewController();

// 打开Web类型对话框
HintDialog.open({
  title: 'Web内容展示',
  content: 'https://gitee.com/qincji/hint-dialog',
  isWebType: true,
  controller: this.webController,
  showNo: true,
  onOk: () => {
    // 获取当前URL
    this.webController.getURL().then(url => {
      console.log('当前网页地址: ' + url);
    });
  }
})
```

#### 2. 事件处理与异步操作

##### 2.1 异步回调处理
对话框按钮支持异步回调操作，可实现复杂业务逻辑：
```typescript
HintDialog.open({
  title: '异步操作示例',
  content: '确定要提交表单吗？',
  onOk: async () => {
    // 显示加载中状态
    HintDialog.open({
      content: '提交中...',
      showNo: false
    });

    try {
      // 模拟API请求
      await new Promise(resolve => setTimeout(resolve, 2000));
      // 关闭加载对话框
      HintDialog.close();
      // 显示成功提示
      HintDialog.open({
        content: '提交成功',
        showNo: false
      });
    } catch (e) {
      HintDialog.close();
      HintDialog.open({
        content: '提交失败，请重试',
        showNo: false
      });
    }
  }
})
```

##### 2.2 生命周期事件
从API version 19开始，自定义弹出框提供生命周期函数：
```typescript
HintDialog.initDefaultStyle({
  // 弹出框显示前回调
  onWillAppear: () => {
    console.log('对话框即将显示');
  },
  // 弹出框显示后回调
  onDidAppear: () => {
    console.log('对话框已显示');
  },
  // 弹出框消失前回调
  onWillDisappear: () => {
    console.log('对话框即将消失');
  },
  // 弹出框消失后回调
  onDidDisappear: () => {
    console.log('对话框已消失');
  }
})
```
[官方指引](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-common-components-custom-dialog)

#### 3. 自定义样式与主题

##### 3.1 全局样式初始化
通过initDefaultStyle方法可全局配置对话框样式：
```typescript
// 导入相关类
import { HintDialog, HintStyle, TitleParm } from '@qincji/dialog';
import { AttributeModifier, TextAttribute, ColumnAttribute } from '@ohos.ui';

// 自定义标题样式
const titleAttr: AttributeModifier<TextAttribute> = {
  applyNormalAttribute(instance: TextAttribute): void {
    instance.fontColor('#ff1145e5')
           .fontWeight(FontWeight.Bolder)
           .fontSize(25)
           .margin({ bottom: 15 });
  }
};

// 自定义外层容器样式
const outBoxAttr: AttributeModifier<ColumnAttribute> = {
  applyNormalAttribute(instance: ColumnAttribute): void {
    instance.backgroundColor('#ffc8f1b3')
           .borderRadius(15)
           .padding(20)
           .margin(50);
  }
};

// 初始化全局样式
HintDialog.initDefaultStyle({
  titleParm: new TitleParm('自定义标题', titleAttr),
  outBoxAttr: outBoxAttr,
  alignment: DialogAlignment.Bottom,
  outsideCancel: true
})
```

##### 3.2 按钮样式定制
通过btnParm参数自定义按钮样式：
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
    space: 15 // 按钮间距
  }
})
```

#### 4. 复杂内容嵌入技巧

##### 4.1 嵌入自定义组件
通过content参数支持自定义组件嵌入：
```typescript
// 定义自定义内容组件
@Component
struct CustomContent {
  @State count: number = 0;

  build() {
    Column({
      space: 10
    }) {
      Text('这是一个自定义计数器')
      Button('+')
        .onClick(() => this.count++)
      Text(`当前值: ${this.count}`)
    }
    .padding(10)
  }
}

// 在对话框中使用
HintDialog.open({
  title: '自定义内容示例',
  content: '', // 必须提供，可空字符串
  customContent: () => CustomContent(), // 自定义内容组件
  showNo: true
})
```

##### 4.2 表单数据收集
利用自定义内容实现表单收集：
```typescript
@State username: string = '';
@State password: string = '';

HintDialog.open({
  title: '用户登录',
  customContent: () => Column({
    space: 15
  }) {
    TextInput({
      placeholder: '请输入用户名'
    })
    .onChange(value => this.username = value)
    .width('100%')

    TextInput({
      placeholder: '请输入密码',
      type: InputType.Password
    })
    .onChange(value => this.password = value)
    .width('100%')
  }
  .padding(10),
  onOk: () => {
    // 处理登录逻辑
    console.log(`登录信息: ${this.username}, ${this.password}`);
  }
})
```

#### 5. 核心源码架构分析

hint-dialog的核心架构采用单例模式设计，确保全局只有一个对话框实例：

```typescript
// 核心实现简化版
class HintDialog {
  private static instance: HintDialog;
  private controller: CustomDialogController;
  private style: HintStyle = defaultStyle;

  // 单例模式
  private constructor() {
    this.initController();
  }

  static getInstance(): HintDialog {
    if (!HintDialog.instance) {
      HintDialog.instance = new HintDialog();
    }
    return HintDialog.instance;
  }

  // 初始化对话框控制器
  private initController() {
    this.controller = new CustomDialogController({
      builder: this.buildContent,
      style: this.style
    });
  }

  // 打开对话框
  static open(params: HintParm) {
    const instance = HintDialog.getInstance();
    instance.updateParams(params);
    instance.controller.open();
  }

  // 其他方法...
}
```

这种设计的优势在于：
1. 避免多个对话框同时弹出的问题
2. 统一管理对话框状态
3. 简化API调用，无需创建实例
4. 便于全局样式配置

#### 6. 与官方Dialog方案对比

| 特性 | hint-dialog | 官方CustomDialog |
|------|-------------|------------------|
| 使用复杂度 | 简单（静态方法调用） | 中等（需创建控制器） |
| 样式定制 | 丰富API支持 | 需手动实现 |
| 功能完整性 | 内置常用功能 | 基础功能需扩展 |
| 代码量 | 少（一行调用） | 多（需定义组件和控制器） |
| 灵活性 | 高（支持自定义内容） | 高（完全自定义） |
| 学习成本 | 低 | 中 |

官方CustomDialog的基本用法示例：
```typescript
// 官方CustomDialog实现
@CustomDialog
struct OfficialDialog {
  controller: CustomDialogController;
  message: string;

  build() {
    Column() {
      Text(this.message)
      Button('确定')
        .onClick(() => this.controller.close())
    }
  }
}

// 使用时需要创建控制器
private dialogController: CustomDialogController = new CustomDialogController({
  builder: OfficialDialog({
    message: '官方对话框',
    controller: this.dialogController
  })
});

// 调用显示
this.dialogController.open();
```

#### 7. 小结
本文详细介绍了hint-dialog的高级用法和定制化技巧，包括参数配置、事件处理、样式定制、复杂内容嵌入等方面。通过源码分析，我们了解了其单例模式的设计思想，并与官方方案进行了对比。hint-dialog在保持灵活性的同时提供了更简洁的API，适合快速开发。下一篇文章将探讨hint-dialog在实际项目中的最佳实践和性能优化策略。