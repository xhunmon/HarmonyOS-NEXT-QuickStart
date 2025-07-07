# 第7篇：常用工具篇（上）

在鸿蒙应用开发中，合理封装和使用工具类可以极大提升开发效率和代码质量。本篇将系统梳理项目中常用的工具类，包括设备唯一ID生成、全局对象缓存、页面跳转、数据持久化等，所有API和术语均严格参考[华为开发者官网](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/development-intro-api)。

## 一、设备唯一ID生成 —— DeviceUtil

设备唯一标识在用户行为分析、数据隔离等场景中非常重要。推荐优先使用`@ohos.security.asset`能力生成持久化ID（卸载重装不变），如不可用则降级为AAID。

- 权限配置：
```json
{
  "name": "ohos.permission.STORE_PERSISTENT_DATA",
  ...
}
```
- 主要实现思路：
  1. 通过`asset.AssetStoreKit`查询或生成唯一ID，并持久化。
  2. 若不支持，则用`AAID.getAAID()`，但卸载/恢复出厂设置后会变化。
- 典型调用：
```ts
const deviceId = await DeviceUtil.getDeviceID();
```

## 二、全局对象缓存 —— global

全局对象缓存工具类可用于存储如UI上下文、函数等对象，便于跨模块调用。还可实现防止按钮快速重复点击等功能。

- 典型用法：
```ts
global.setObject('UI_CONTEXT', getContext(this));
const uiContext = global.getObject<common.UIAbilityContext>('UI_CONTEXT');
Button('登录').onClick(global.blockQuickClick(() => { /* 登录逻辑 */ }))
```
- 相关API：[全局对象与上下文管理](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-app-context-V5)

## 三、页面跳转工具 —— IntentUtil

页面跳转和三方应用唤起是移动开发的常见需求。IntentUtil工具类支持：
- 跳转WebView页面：`IntentUtil.jumpWeb(url, title)`
- 跳转应用市场：`IntentUtil.jumpStore(appId)`
- 跳转其他APP：`IntentUtil.jumpApp(deviceId, bundleName, abilityName)`
- 隐式跳转（如打开文档）：`IntentUtil.jumpAction(action, entities, uri, type)`
- 智能解析跳转：`IntentUtil.jumpUrl(url或json)`

- 相关API：[AbilityKit能力调用](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/ability-start-ability-V5)

```ts
/*
 * @Desc: 
 * @Author: qincji
 * @Date: 2024/6/8
 */
import { common, Want } from '@kit.AbilityKit';
import { NavUtil } from 'common_ui';
import { global } from './Global';

export class IntentUtil {

  static jumpWeb(url: string, title: string = '') {
    const record: Record<string, Object> = { 'url': url, 'title': title };
    NavUtil.push('WebPage', record)
  }


  static jumpApp(bundleName: string, abilityName: string, deviceId?: string) {
    let want: Want = {
      deviceId: deviceId,
      bundleName: bundleName,
      abilityName: abilityName,
    };

    IntentUtil.jumpWant(want)
  }


  static jumpAction(action: string, entities: string, uri: string, type: string) {
    let want: Want = {
      action: action,
      entities: [entities], // entities can be omitted
      uri: uri,
      type: type,
    };

    IntentUtil.jumpWant(want)
  }


  static jumpBrows(url: string) {
    let want: Want = {
      action: 'ohos.want.action.viewData',
      entities: ['entity.system.browsable'],
      uri: url
    }
    IntentUtil.jumpWant(want)
  }

  static jumpStore(appId: string) {
    const want: Want = {
      uri: `store://appgallery.huawei.com/app/detail?id=${appId}`
    };
    IntentUtil.jumpWant(want)
  }

  static jumpSettingsApp(bundleName: string) {
    let want: Want = {
      bundleName: 'com.huawei.hmos.settings',
      abilityName: 'com.huawei.hmos.settings.MainAbility',
      uri: 'application_info_entry',
      parameters: { pushParams: bundleName }
    };
    IntentUtil.jumpWant(want);
  }

  static jumpWant(want: Want) {
    global.getObject<common.UIAbilityContext>('ui_context').startAbility(want)
  }

  static jumpUrl(url: string) {
    if (!url) {
      return;
    }
    if (url.startsWith('http://') || url.startsWith('https://')) {
      IntentUtil.jumpBrows(url);
      return;
    }
    const data: Record<string, string> = JSON.parse(url);
    if (data === undefined) {
      return;
    }
    const appId: string = data['appId'];
    if (appId !== undefined) {
      IntentUtil.jumpStore(appId);
      return;
    }

    //{"deviceId":"","bundleName":"cn.qincji.zerooneapp","abilityName":"EntryAbility"}
    const deviceId: string = data['deviceId'];
    const bundleName: string = data['bundleName'];
    const abilityName: string = data['abilityName'];

    if (bundleName !== undefined && abilityName !== undefined) {
      IntentUtil.jumpApp(bundleName, abilityName, deviceId);
      return;
    }

    //{"action":"ohos.want.action.search","entities":"entity.system.browsable","uri":"https://www.test.com:8080/query/student","type":"text/plain"}
    const action: string = data['action'];
    const entities: string = data['entities'];
    const uri: string = data['uri'];
    const type: string = data['type'];

    if (action !== undefined && entities !== undefined && uri !== undefined && type !== undefined) {
      IntentUtil.jumpAction(action, entities, uri, type);
      return;
    }
  }
}

```

## 四、数据持久化（4M以下）—— kvManger

适用于存储较大数据（如base64图片），基于`distributedKVStore`实现，支持本地和多设备协同。
- 初始化：`kvManger.init(this.context)`
- 存取删：`put(key, value)`、`get(key, defVal)`、`delete(key)`
- 相关API：[分布式KV存储](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-data-persistence-kvstore-V5)

## 五、首选项数据持久化（几K以下）—— pref

适合存储轻量级配置（如用户偏好、设置项），基于`@ohos.data.preferences`实现。
- 初始化：`pref.init(this.context)`
- 存取删：`put(key, value)`、`get(key, defVal)`、`delete(key)`
- 相关API：[首选项存储](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-data-persistence-preferences-V5)

## 六、最佳实践与官方文档

- 工具类建议单一职责、接口简洁，便于复用和维护。
- 重要能力如设备ID、数据存储等务必参考[官方文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/overview-V5)和[ArkTS编程规范](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-coding-style-guide-V5)。
- 代码中应有详细注释，关键异常需日志记录，便于排查。

**到这里，所有的内容已经结束了！本章的完整源码已经上传到gitee了：[鸿蒙应用0-1开发](https://gitee.com/qincji/ZeroOneApp)。**