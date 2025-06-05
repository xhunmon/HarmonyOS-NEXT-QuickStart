# Complete Guide to Obtaining and Configuring Signing Files

Before publishing a HarmonyOS Next app, developers must configure valid signing files, including the developer certificate (.p12) and Profile file. Signing files ensure app security and are required for platform review. This article details the application and configuration process for signing files, common issues, and official resources to help developers complete signing configuration smoothly.

## 1. Purpose of Signing Files

- **Identity Authentication**: Signing files prove the developer's identity and prevent app tampering or forgery.
- **Platform Review**: All apps published on AppGallery Connect must be signed; unsigned or invalidly signed apps will be rejected.
- **Security Enhancement**: Signing prevents repackaging or malicious code injection.

## 2. [Signing File Application Process](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ide-publish-app#section793484619307)

1. **Log in to AppGallery Connect**
   - Visit [AppGallery Connect](https://developer.huawei.com/consumer/cn/service/josp/agc/index.html) and log in with your developer account.

2. **Go to "My Certificates"**
   - In the console's left navigation, select "Project Settings > My Certificates."

3. **Apply for Developer Certificate (.p12)**
   - Fill in the required info as instructed, generate and download the .p12 certificate.
   - Set a certificate password and keep it safe.

4. **Generate Profile File**
   - In "Profile Management," select "New Profile," bind the app package name and device info.
   - Download the Profile file and save it locally.

> **Tip**: Certificates and Profile files are sensitive; back them up securely to avoid leaks or loss.

## 3. Signing File Configuration Method

1. **Place .p12 and Profile Files in Project Directory**
   - Recommended to store in the `signature/` folder for easy management.

2. **Configure build-profile.json5**
   - In DevEco Studio's build-profile.json5, set the signing path and password (enter password in IDE as per the signing file guide):
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

3. **Profile File Configuration**
   - In DevEco Studio's "Project Settings," select the corresponding Profile file, ensuring it matches the app package name and device info.

4. **Packaging Verification**
   - Use "Build > Build Release APP" to generate the installable package and check the signing info.

## 4. Common Issues and Tips

- **Lost Certificate or Forgotten Password**: Reapply for a certificate and update the project to avoid packaging issues.
- **Profile File Mismatch**: If the Profile file doesn't match the package name, packaging or review will fail; always verify info.
- **Expired or Revoked Certificate**: Apply for a new certificate in AppGallery Connect and update the project.
- **Confused Certificate Management in Teamwork**: Manage signing files centrally to avoid version confusion.
- **Release and Test p12**: The same file can be used for both.

## 5. Official Reference Links

- [AppGallery Connect Certificate Management](https://developer.huawei.com/consumer/cn/service/josp/agc/index.html#/harmonyOSDevPlatform/9249519184596237889)
- [HarmonyOS Next Official Docs](https://developer.huawei.com/consumer/cn/doc/)
- [DevEco Studio Official Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ide-quick-start)
- [Huawei Developer Alliance Account Center](https://developer.huawei.com/consumer/cn/)

## 6. Summary

Obtaining and configuring signing files is a key step before publishing a HarmonyOS Next app. Strictly follow the official process to apply for and manage certificates, regularly check Profile file consistency, and ensure smooth packaging and review. For signing issues, consult official docs or the community to ensure app security and compliance. 