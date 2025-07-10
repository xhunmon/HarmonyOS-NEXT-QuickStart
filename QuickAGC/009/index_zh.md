# 上传包体与资质材料全流程指南

在AppGallery Connect（AGC）平台成功创建应用后，开发者需要上传应用安装包（.app文件）和相关资质材料。规范、完整地上传包体和材料，是应用顺利通过审核、成功上架的关键。本文将详细介绍包体上传流程、资质材料准备、常见问题及官方参考资源，帮助开发者高效完成上架准备。

## 一、包体准备与签名

### Hvigor打包文件
hvigor-config.json5文件整体的结构如下：
```bash
modelVersion
dependencies
execution
└── analyze
└── daemon
└── incremental
└── parallel
└── typeCheck
└── optimizationStrategy
logging
└── level
debugging
└── stacktrace
nodeOptions
└── maxOldSpaceSize
└── maxSemiSpaceSize
└── exposeGC
javaOptions
└── Xmx
properties
└── hvigor.cacheDir
└── ohos.buildDir
└── enableSignTask
└── ohos.arkCompile.maxSize
└── hvigor.pool.cache.capacity
└── hvigor.pool.maxSize
└── ohos.pack.compressLevel
└── hvigor.analyzeHtml
└── hvigor.dependency.useNpm
└── ohos.compile.lib.entryfile
└── ohos.align.target
└── ohos.fallback.target
└── ohos.arkCompile.sourceMapDir
└── ohos.collect.debugSymbol
└── hvigor.enableMemoryCache
└── hvigor.memoryThreshold
└── ohos.nativeResolver
└── ohos.arkCompile.noEmitJs
└── ohos.arkCompile.singleFileEmit
└── ohos.sign.har
└── hvigor.keepDependency
└── ohos.arkCompile.emptyBundleName
└── ohos.uiTransform.Optimization
└── ohos.har.excludeHspDependencies
└── ohos.processLib.optimization
└── ohos.obfuscationRules.optimization
└── hvigor.incremental.optimization
└── hvigor.task.schedule.optimization
└── ohos.byteCodeHar.integratedOptimization
└── ohos.rollupCache.usePathPlaceholder
└── ohos.rollupCache.useSourceHash
```

### 签名配置检查
在`build-profile.json5`中验证签名配置：
```json
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


1. **生成签名安装包**
   - 在DevEco Studio中完成打包与签名，生成`.app`格式的安装包。
   - 检查包体签名、包名、版本号等信息，确保与AGC应用信息一致。

2. **登录AppGallery Connect**
   - 访问[AppGallery Connect](https://developer.huawei.com/consumer/cn/service/josp/agc/index.html)，进入"我的应用"。

3. **进入"版本管理"页面**
   - 选择目标应用，点击"版本管理"或"版本信息"菜单。

4. **上传安装包**
   - 点击"上传安装包"，选择本地`.app`文件上传。
   - 填写版本号、更新日志等信息。
   - 系统会自动进行包体检测，提示签名、包名、权限等合规性问题。

5. **保存并提交审核**
   - 检查所有信息无误后，点击"保存"并提交审核。

## 二、[资质材料准备](https://developer.huawei.com/consumer/cn/doc/app/agc-help-harmonyos-releaseapp-0000001914554900#section12761327123615)

1. **软著证书**
   - 上传《软件著作权登记证书》或《电子版权认证》扫描件，确保信息清晰、真实。
   - 可以自己办理但是时间较长，可以拖第三方办理，淘宝300RMB就左右可以办下来了。
   
2. **隐私政策链接**
   - 提供独立的隐私政策页面链接，内容需与应用实际一致。

3. **应用截图**
   - 上传展示核心功能的高质量截图，按平台要求提供不同尺寸。

4. **其他材料**
   - 如涉及支付、地图、推送等特殊功能，需上传相关资质证明或授权文件。

## 三、常见问题与避坑指南

- **包体签名不符**：包体签名与AGC证书不一致会导致审核失败，务必核对签名信息。
- **包名或版本号错误**：包名、版本号需与AGC应用信息完全一致。
- **资质材料不全或模糊**：上传材料需清晰、真实，缺失或不清晰会被驳回。
- **隐私政策内容不符**：隐私政策需与应用实际功能、权限一致。
- **特殊功能未提供资质**：如涉及支付、地图等，需提前准备相关资质证明。

## 四、官方参考链接

- [AppGallery Connect包体上传与审核指引](https://developer.huawei.com/consumer/cn/doc/app/agc-help-harmonyos-releaseapp-0000001914554900#section1754010232299)
- [HarmonyOS Next官方文档](https://developer.huawei.com/consumer/cn/doc/)
- [华为开发者联盟账号中心](https://developer.huawei.com/consumer/cn/)

## 五、总结

上传包体与资质材料是HarmonyOS Next应用上架流程的关键环节。建议开发者提前准备好所有必需材料，严格按照官方要求上传，遇到疑问时优先查阅官方文档或在社区发帖求助，确保应用顺利通过审核并成功上架。
