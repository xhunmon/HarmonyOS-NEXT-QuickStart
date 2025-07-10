# Detailed Guide to Packaging and Signing Process

After completing the development and testing of a HarmonyOS Next application, it needs to be packaged and signed to generate an installable package for上架. The correct packaging and signing process is an important prerequisite for the application to pass the review smoothly and ensure security. This article will detail the packaging process, signature file configuration, common issues, and official reference resources for HarmonyOS Next applications, helping developers efficiently complete the release preparation.

## 1. Overview of Packaging Process

1. **Prepare Signature Files**
   
   - You need to apply for and prepare the developer certificate (.p12 file) and Profile file in advance. For the creation process, refer to the official website: [How to Package with Manually Generated Certificates](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ide-signing#section297715173233).
   - Certificates can be applied for through AppGallery Connect, and the Profile file is used to bind the application package name and device information.
   
2. **Configure Signature Information**
   
   - Configure the signature path and password in the build-profile.json5 file of the DevEco Studio project (you need to enter the password using the IDE, refer to preparing signature files):
   
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

   - Ensure the signature file path and password are filled in correctly to avoid packaging failures.
   
3. **Generate Release Package**
   - In DevEco Studio, select "Build > Build Hap(s)/APP(s) > Build Release APP".
   - The system will automatically package and sign, generating an installation package in ".app" format.
   - Check the output directory to confirm the generated installation package and signature information are correct.

## 2. Obtaining and Managing Signature Files

- **Certificate Application**: Log in to AppGallery Connect, enter the "My Certificates" page, and follow the instructions to apply for a developer certificate (.p12) and Profile file.
- **Certificate Management**: Safely store certificates and passwords to avoid leakage. It is recommended to back them up to a secure location.
- **Profile File Update**: If the application package name or device information changes, regenerate the Profile file and update it in the project.

```json
// Configure application information app.json5:
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
```

## 3. Common Issues and Pitfalls

- **Lost Signature Files or Incorrect Passwords**: Will cause packaging failures or inability to install the application. Be sure to safely store certificates and passwords.
- **Profile File Not Updated**: Failure to synchronize the Profile after changing the package name or device information may result in review rejection.
- **Not Checking Signature Information After Packaging**: It is recommended to verify the installation package signature using official tools or command lines to ensure correctness.
- **Expired or Revoked Certificates**: Timely apply for new certificates in AppGallery Connect and update them in the project.

## 4. Official Reference Links

- [HarmonyOS Next Official Documentation](https://developer.huawei.com/consumer/cn/doc/)
- [DevEco Studio Official Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ide-quick-start)
- [AppGallery Connect Certificate Management](https://developer.huawei.com/consumer/cn/service/josp/agc/index.html#/myApp)
- [Huawei Developer Alliance Account Center](https://developer.huawei.com/consumer/cn/)

## 5. Summary

Packaging and signing are key steps before releasing a HarmonyOS Next application. It is recommended that developers strictly follow the official process to prepare and manage signature files, regularly check certificate validity, and ensure the generated installation package is secure and compliant. When encountering packaging or signing issues, prioritize consulting official documentation or posting for help in the community to ensure the application passes the review smoothly and goes online securely.