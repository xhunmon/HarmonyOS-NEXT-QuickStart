# 审核发布与常见问题全流程解析

在完成包体和资质材料上传后，HarmonyOS Next应用将进入AppGallery Connect（AGC）平台的审核与发布环节。审核通过后，应用才能正式上线。本文将详细介绍审核流程、发布注意事项、常见驳回原因、问题应对策略及官方参考资源，帮助开发者顺利完成上架最后一步。

## 一、审核流程概述

1. **提交审核**
   - 在AGC平台确认所有信息无误后，点击"提交审核"。

2. **初步自动检测**
   - 系统自动检测包体签名、包名、权限、资质材料等合规性。
   - 检测通过后进入人工审核环节。

3. **人工审核**
   - 审核员对应用功能、界面、合规性、隐私政策等进行全面检查。
   - 重点关注用户协议、隐私政策、权限申请、功能稳定性等。

4. **审核结果通知**
   - 审核通过：应用进入发布状态，自动上线AppGallery。
   - 审核驳回：平台会给出驳回原因，需根据反馈修改后重新提交。

## 二、发布注意事项

- **审核周期**：一般为3-7个工作日，复杂应用或材料补充可能延长。
- **灰度发布**：建议通过灰度发布功能，先小范围上线，验证兼容性和用户反馈。
- **多端协同**：如适配多设备，需分别测试并声明支持的设备类型。
- **后续更新**：每次版本更新需重新提交审核，建议提前规划测试和材料准备。

## 三、常见驳回原因

- **隐私政策缺失或内容不符**：未上传或内容与实际功能不符。
- **权限说明不清晰**：权限弹窗未明确说明用途。
```typescript
//检查该文件，并且查看 AGC 是否申请一些额外的危险权限
"requestPermissions": [
      {
        "name": "ohos.permission.INTERNET"
      },
      {
        "name": "ohos.permission.GET_NETWORK_INFO"
      },
      {
        "name": "ohos.permission.KEEP_BACKGROUND_RUNNING",
        "reason": "$string:permission_audio_record_background",
        "usedScene": {
          "abilities": [
            "EntryAbility"
          ],
          "when": "always"
        }
      },
      {
        "name": "ohos.permission.MICROPHONE",
        "reason": "$string:permission_audio_record",
        "usedScene": {
          "abilities": [
            "EntryAbility"
          ],
          "when": "inuse"
        }
      },
      {
        "name": "ohos.permission.STORE_PERSISTENT_DATA",
        "reason": "$string:permission_read_image",
        "usedScene": {
          "abilities": [
            "EntryAbility"
          ],
          "when": "inuse"
        }
      },
      {
        "name": "ohos.permission.CAMERA",
        "reason": "$string:permission_phone",
        "usedScene": {
          "abilities": [
            "EntryAbility"
          ],
          "when": "inuse"
        }
      }
    ]
```

- **应用闪退、卡顿或功能异常**：基础功能不稳定。
- **资质材料不全或不清晰**：如软著证书、隐私政策、截图等不合规。
- **包名、签名、版本号不一致**：与AGC信息不符。

```typescript
//配置应用信息app.json5：
{
  "app": {
    "bundleName": "com.xxx.xxx",
    "vendor": "demo",
    "versionCode": 1010000,
    "versionName": "1.1.0",
    "icon": "$media:app_icon",
    "label": "$string:app_name"
  }
}

//build-profile.json5
{
  "app": {
    "signingConfigs": [
      {
        "name": "default",
        "type": "HarmonyOS",
        "material": {
          "storeFile": "xxx.p12",
          "storePassword": "xxx",
          "keyAlias": "xxx",
          "keyPassword": "xxx",
          "signAlg": "SHA256withECDSA",
          "profile": "xxx.p7b",
          "certpath": "xxx.cer"
        }
      },
    ],
    "products": [
      {
        "name": "default",
        "signingConfig": "default",
        "compatibleSdkVersion": "5.0.0(12)",
        "runtimeOS": "HarmonyOS",
        "buildOption": {
          "strictMode": {
            "useNormalizedOHMUrl": true
          }
        }
      },
    "buildModeSet": [
      {
        "name": "debug",
      },
      {
        "name": "release"
      }
    ]
  },
  "modules": [
    {
      "name": "entry",
      "srcPath": "./entry",
      "targets": [
        {
          "name": "default",
          "applyToProducts": [
            "default"
          ]
        }
      ]
    },
  ]
}
```

- **认真阅读驳回原因**：根据平台反馈逐条排查和修正。
- **补充和完善材料**：确保所有必需材料真实、清晰、合规。
- **优化功能和体验**：修复闪退、卡顿等问题，提升应用稳定性。
- **多设备真机测试**：确保所有声明支持的设备均能正常运行。
- **主动沟通**：如遇审核疑难，可通过AGC平台工单或开发者社区寻求帮助。
- **解决方案**：每次驳回详细问题可能描述的不清楚，最好加鸿蒙开发者助手，让他帮忙反馈。

## 五、官方参考链接

- [AppGallery Connect审核与发布官方指引](https://developer.huawei.com/consumer/cn/doc/app/agc-help-harmonyos-releaseapp-0000001914554900)
- [HarmonyOS Next官方文档](https://developer.huawei.com/consumer/cn/doc/)
- [华为开发者联盟账号中心](https://developer.huawei.com/consumer/cn/)

## 六、总结

审核发布是HarmonyOS Next应用上架流程的最后一步。建议开发者严格按照官方要求准备材料和功能，遇到驳回及时查阅反馈并修正，持续优化产品质量。通过科学的发布策略和积极的沟通，确保应用顺利上线并获得用户认可。
