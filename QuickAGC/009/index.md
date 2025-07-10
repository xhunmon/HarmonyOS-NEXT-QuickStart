# Uploading Application Packages and Qualification Materials for HarmonyOS Next

After successfully creating an application on AppGallery Connect (AGC), developers need to upload the application installation package (.app file) and relevant qualification materials. Standardized and complete uploads are crucial for smooth review and successful publication. This guide details the package upload process, qualification material preparation, common issues, and official resources to help developers efficiently complete publication preparations.

## 1. Package Preparation and Signing

### Hvigor Packaging File
The structure of the `hvigor-config.json5` file is as follows:
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

### Signing Configuration Verification
Verify signing configuration in `build-profile.json5`:
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

1. **Generate Signed Installation Package**
   - Complete packaging and signing in DevEco Studio to generate `.app` installation package
   - Verify package signature, name, and version number match AGC application information

2. **Log in to AppGallery Connect**
   - Access AppGallery Connect → "My Apps"

3. **Navigate to "Version Management" Page**
   - Select target application → Click "Version Management" or "Version Information"

4. **Upload Installation Package**
   - Click "Upload Installation Package" → Select local `.app` file
   - Fill in version number, update logs, etc.
   - System automatically performs package detection and flags compliance issues

5. **Save and Submit for Review**
   - Verify all information → Click "Save" → Submit for review

## 2. Preparation of Qualification Materials

1. **Software Copyright Certificate**
   - Upload scanned copy of "Software Copyright Registration Certificate" or "Electronic Copyright Certification"
   - Can be self-processed (time-consuming) or through third-party services (≈300 RMB)

2. **Privacy Policy Link**
   - Provide independent privacy policy page URL with content matching actual app functionality

3. **Application Screenshots**
   - Upload high-quality screenshots showcasing core features in required dimensions

4. **Other Materials**
   - For special features (payment, maps, push notifications), provide relevant authorization documents

## 3. Common Issues & Solutions

| Issue | Solution |  
|-------|----------|  
| **Signature mismatch** | Verify signing configuration matches AGC certificate |  
| **Incorrect package name/version** | Ensure exact match with AGC application information |  
| **Incomplete/blurred materials** | Provide clear, authentic scanned documents |  
| **Privacy policy inconsistency** | Align policy content with actual app functionality |  
| **Missing special feature authorization** | Prepare relevant certificates in advance |  

## 4. Official References

- AppGallery Connect Package Upload & Review Guide
- HarmonyOS Next Documentation
- Huawei Developer Alliance Account Center

## 5. Conclusion

Uploading packages and qualification materials is critical for HarmonyOS Next app publication. Developers should:
1. Prepare all required materials in advance
2. Strictly follow official requirements during upload
3. Consult documentation and communities for complex issues
4. Double-check signatures and package information  
   This ensures smooth review approval and successful app publication.