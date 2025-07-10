# 第1篇：鸿蒙APP开发怎么样开始？

鸿蒙操作系统（HarmonyOS）是华为自主研发的新一代智能终端操作系统，凭借其跨设备无缝协作、高性能和安全性，吸引了大量开发者关注。随着鸿蒙生态的不断壮大，越来越多的开发者希望加入其中，开发属于自己的鸿蒙应用（APP）。本篇文章将详细介绍如何从零开始准备鸿蒙APP开发，帮助你顺利迈出第一步。

## 一、开发准备

### 1.1 开发者权限
目前鸿蒙开发已面试所有开发者。

### 1.2 设备和系统要求
- 一台运行Windows、macOS或Linux的电脑，建议内存16GB及以上。
- 安装[DevEco Studio 下载需要登陆华为账号](https://developer.huawei.com/consumer/cn/download/deveco-studio)，这是官方推荐的鸿蒙应用开发工具。
- 推荐使用真机（如Mate 60及以上机型），如无真机可使用[官方模拟器](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-emulator-create-V5)， IDE自带。

> 注意：Mac Intel芯片基本不支持模拟器的使用，刷成windows系统则可以使用。

### 1.3 基础编程语言
鸿蒙APP NEXT支持多种编程语言，但目前最推荐使用ArkTS（基于TypeScript的增强型语言）。建议先学习：
- JavaScript高级程序设计
- [TypeScript官方文档](https://www.tslang.cn/docs/home.html)
- [ArkTS官方文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/learning-arkts-V5)

如果你有其他编程语言基础，可以直接上手ArkTS，遇到不懂的再回头补充相关知识。

### 1.4 API版本选择
建议从API 12版本开始学习和开发，以往版本可以完全忽略，因为系统推出基本是从5.0（对应API 12）开始。

## 二、[创建第一个ArkTS应用](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/start-with-ets-stage-V5)

建议选择Stage模型进行开发，这是官方推荐的应用架构。开发过程中需要熟悉以下关键配置文件和目录结构：

- AppScope > app.json5：应用全局配置信息
- entry：HarmonyOS工程模块，编译生成HAP包
    - src > main > ets：存放ArkTS源码
    - src > main > ets > entryability：应用/服务入口
    - src > main > ets > pages：应用/服务页面
    - src > main > resources：资源文件（图片、多媒体、字符串、布局等）
    - src > main > module.json5：模块配置文件，包含HAP包和设备配置信息
    - build-profile.json5：模块编译信息配置
    - hvigorfile.ts：模块级编译构建任务脚本
    - obfuscation-rules.txt：混淆规则文件，保护代码安全
- oh_modules：三方库依赖信息
- build-profile.json5：应用级配置信息，包括签名、产品配置等
- hvigorfile.ts：应用级编译构建任务脚本

```ts
//demo Index.ets
import { BusinessError } from '@kit.BasicServicesKit';


@Entry
@Component
struct Index {
  @State message: string = 'Hello World';


  build() {
    Row() {
      Column() {
        Text(this.message)
          .fontSize(50)
          .fontWeight(FontWeight.Bold)
        Button() {
          Text('Next')
            .fontSize(30)
            .fontWeight(FontWeight.Bold)
        }
        .type(ButtonType.Capsule)
        .margin({
          top: 20
        })
        .backgroundColor('#0D9FFB')
        .width('40%')
        .height('5%')
        .onClick(() => {
          console.info(`Succeeded in clicking the 'Next' button.`)
          let uiContext: UIContext = this.getUIContext();
          let router = uiContext.getRouter();
          router.pushUrl({ url: 'pages/Second' }).then(() => {
            console.info('Succeeded in jumping to the second page.')
          }).catch((err: BusinessError) => {
            console.error(`Failed to jump to the second page. Code is ${err.code}, message is ${err.message}`)
          })
        })
      }
      .width('100%')
    }
    .height('100%')
  }
}
```

开发小贴士：
- 在DevEco Studio编辑窗口右上角点击Previewer可预览页面，Ctrl+S可实时刷新。
- 页面跳转推荐使用[Navigation](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-navigation-navigation-V5)而非传统router。
- 使用模拟器可以直接安装运行。
- 使用真机安装则会比较麻烦，首先[使用usb或WiFi连接上](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-run-device-V5)，然后[对设备进行签名](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ide-signing-V5)。

## 三、如何快速入手？

- 1. 掌握[ArkTS](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-V5)
- 2. 了解[UI范式基本语法](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-ui-paradigm-basic-syntax-V5)
- 3. [状态管理](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-state-management-V5)，推荐V2
- 4. [然后可以开发UI了](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-ui-development-V5)



下一篇文章将带你正式搭建鸿蒙项目，敬请期待！
