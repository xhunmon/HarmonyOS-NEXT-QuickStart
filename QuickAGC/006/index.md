# Detailed Guide to Packaging and Signing Process

After development and testing, HarmonyOS Next apps need to be packaged and signed to generate an installable package for publishing. A correct packaging and signing process is essential for passing review and ensuring security. This article details the packaging process, signing file configuration, common issues, and official resources to help developers efficiently prepare for release.

## 1. Packaging Process Overview

1. **Prepare Signing Files**
   - Obtain the developer certificate (.p12) and Profile file in advance. See the official guide: [How to Manually Generate Certificates for Packaging](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ide-signing#section297715173233).
   - Certificates can be applied for via AppGallery Connect; Profile files bind the app package name and device info.
2. **Configure Signing Info**
   - In the DevEco Studio project's build-profile.json5, configure the signing path and password (enter password in IDE as per the signing file guide):
   ```json
   "app": {
       "signingConfigs": [
         {
           "name": "default",
           "type": "HarmonyOS",
           "material": {
             "storeFile": "doc/debug/debug.p12",
             "storePassword": "xxxxxxxxxxxx",
             "keyAlias": "debug",
             "keyPassword": "xxxxxxxxxxxx",
             "signAlg": "SHA256withECDSA",
             "profile": "doc/debug/debug.p7b",
             "certpath": "doc/debug/debug.cer"
           }
         }
      ]
   }
   ```
   - Ensure the signing file path and password are correct to avoid packaging failures.
3. **Generate Release Package**
   - In DevEco Studio, select "Build > Build Hap(s)/APP(s) > Build Release APP."
   - The system will automatically package and sign, generating a ".app" installable package.
   - Check the output directory to confirm the package and signing info are correct.

## 2. Obtaining and Managing Signing Files

- **Certificate Application**: Log in to AppGallery Connect, go to "My Certificates," and follow the instructions to apply for a .p12 certificate and Profile file.
- **Certificate Management**: Keep certificates and passwords safe; back them up securely.
- **Profile File Updates**: If the app package name or device info changes, regenerate the Profile and update the project.

## 3. Common Issues and Tips

- **Lost Signing File or Wrong Password**: This will cause packaging failure or app installation issues; keep certificates and passwords safe.
- **Profile Not Updated**: If the package name or device info changes but Profile isn't updated, review may fail.
- **Not Verifying Signing Info After Packaging**: Use official tools or CLI to verify the package signature.
- **Expired or Revoked Certificate**: Apply for a new certificate in AppGallery Connect and update the project.

## 4. Official Reference Links

- [HarmonyOS Next Official Docs](https://developer.huawei.com/consumer/cn/doc/)
- [DevEco Studio Official Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ide-quick-start)
- [AppGallery Connect Certificate Management](https://developer.huawei.com/consumer/cn/service/josp/agc/index.html#/myApp)
- [Huawei Developer Alliance Account Center](https://developer.huawei.com/consumer/cn/)

## 5. Summary

Packaging and signing are key steps before publishing a HarmonyOS Next app. Strictly follow the official process to prepare and manage signing files, regularly check certificate validity, and ensure the installable package is secure and compliant. For packaging or signing issues, consult official docs or the community to ensure smooth review and secure launch. 