# Part 1: How to Start HarmonyOS App Development?

HarmonyOS (HarmonyOS, 华为鸿蒙操作系统) is a next-generation intelligent operating system independently developed by Huawei. With its seamless cross-device collaboration, high performance, and robust security, HarmonyOS has attracted a growing number of developers to its ecosystem. As the ecosystem expands, more developers are eager to join and create their own HarmonyOS applications (Apps). This article provides a detailed guide on how to get started with HarmonyOS app development from scratch, helping you take your first step with confidence.

## 1. Preparation for Development

### 1.1 Developer Access
Currently, HarmonyOS NEXT development is open to all developers. For the latest updates and official activities, please refer to the [Huawei Developer Forum](https://developer.huawei.com/consumer/en/forum/block/harmonyos-next?filterCondition=1).

### 1.2 Device and System Requirements
- A computer running Windows, macOS, or Linux (16GB RAM or above is recommended).
- Install [DevEco Studio (Huawei official IDE, login required)](https://developer.huawei.com/consumer/en/download/deveco-studio), the official IDE for HarmonyOS app development.
- It is recommended to use a real device (such as Mate 60 or above). If you do not have a real device, you can use the [official emulator](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/ide-emulator-create-V5) integrated in DevEco Studio.

> Note: Mac devices with Intel chips are generally not supported by the emulator. You may use Windows OS for better compatibility.

### 1.3 Basic Programming Languages
HarmonyOS NEXT supports multiple programming languages, but it is highly recommended to use [ArkTS (Ark TypeScript)](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/learning-arkts-V5), an enhanced superset of TypeScript. Suggested learning path:
- JavaScript Advanced Programming
- [TypeScript Official Documentation](https://www.tslang.cn/docs/home.html)
- [ArkTS Official Documentation](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/learning-arkts-V5)

If you have experience with other programming languages, you can start directly with ArkTS and refer back to the basics as needed.

### 1.4 API Version Selection
It is recommended to start with API version 12, as previous versions can be ignored. The system is officially promoted from version 5.0 (corresponding to API 12).

## 2. [Create Your First ArkTS Application](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/start-with-ets-stage-V5)

It is recommended to use the Stage model, which is the official application architecture. During development, you should become familiar with the following key configuration files and directory structure:

- AppScope > app.json5: Global application configuration
- entry: HarmonyOS project module, compiled into a HAP package
    - src > main > ets: ArkTS source code
    - src > main > ets > entryability: Application/service entry
    - src > main > ets > pages: Application/service pages
    - src > main > resources: Resource files (images, multimedia, strings, layouts, etc.)
    - src > main > module.json5: Module configuration, including HAP and device info
    - build-profile.json5: Module build configuration
    - hvigorfile.ts: Module-level build script
    - obfuscation-rules.txt: Obfuscation rules for code security
- oh_modules: Third-party library dependencies
- build-profile.json5: Application-level configuration (signing, product info, etc.)
- hvigorfile.ts: Application-level build script

**Development Tips:**
- In DevEco Studio, click the Previewer in the top-right toolbar to preview your page. Use Ctrl+S to refresh in real time.
- For page navigation, use [Navigation](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-navigation-navigation-V5) instead of the traditional router.
- The emulator allows direct installation and running of your app.
- For real devices, connect via [USB or WiFi](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/ide-run-device-V5) and [sign your device](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/ide-signing-V5) before installation.

## 3. How to Get Started Quickly?

1. Master [ArkTS](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-V5)
2. Understand the [UI Paradigm Basic Syntax](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-ui-paradigm-basic-syntax-V5)
3. Learn about [State Management (V2 recommended)](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-state-management-V5)
4. [Start developing your UI](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-ui-development-V5)

The next article will guide you through building a real HarmonyOS project. Stay tuned! 