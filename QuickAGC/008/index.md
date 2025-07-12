# Complete Guide to Creating an App in AppGallery Connect

AppGallery Connect (AGC) is Huawei's one-stop platform for application development, distribution, and operation. Creating and configuring your HarmonyOS Next application in AGC is a critical step before publishing to the HarmonyOS Gallery. This guide walks through the complete process of creating an application in AGC, configuring project files, filling in application information, and linking your development project, ensuring a smooth transition from development to release.

## 1. Prerequisites for Application Creation

Before creating an application in AGC, ensure you have the following:
- A registered [Huawei Developer Account](https://developer.huawei.com/consumer/cn/).
- Completed identity verification (individual or enterprise) in the Huawei Developer Console.
- Basic application information prepared, including application name, supported devices, and description.
- Development environment set up with DevEco Studio and HarmonyOS SDK installed.

## 2. Step-by-Step Application Creation in AGC

### 2.1 Access AppGallery Connect
1. Visit the [AppGallery Connect Console](https://developer.huawei.com/consumer/cn/service/josp/agc/index.html#/myApp) and log in with your Huawei Developer Account.
2. Click **My Projects** to enter the project management page.

### 2.2 Create a New Project
1. Click **New Project** in the upper-right corner.
2. Enter a **Project Name** (e.g., "MyHarmonyApp") and select a **Project Time Zone**.
3. Click **Create** to complete project creation.

### 2.3 Add a HarmonyOS Application to the Project
1. In the project dashboard, click **Add App**.
2. Fill in the application information:
   - **Platform**: Select **HarmonyOS**.
   - **Device**: Choose supported devices (e.g., Phone, Tablet, Smart TV).
   - **App Name**: Enter the application name (must match the name in `app.json5`).
   - **Default Language**: Select the primary language (e.g., English).
   - **Package Name**: Enter a unique package name (e.g., `com.company.appname`), which must match the `bundleName` in `app.json5`.
3. Click **OK** to create the application.

### 2.4 Obtain Application Configuration Information
After creating the application, navigate to **Project Settings > General Information** to获取关键配置信息:
- **App ID**: Unique identifier for the application in AGC.
- **Client Secret**: Required for integrating certain AGC services (e.g., Auth Service).
- **OAuth 2.0 Client ID**: Used for authentication services.
- **AGC Plugin Configuration**: Dependency information for integrating AGC services into the project.

## 3. Configuring Project Files

To link your DevEco Studio project with AGC, configure the following core files:

### 3.1 app.json5
This file defines global application information that must match the AGC configuration:
```json
{
  "app": {
    "bundleName": "com.company.appname", // Must match AGC package name
    "vendor": "CompanyName",
    "versionCode": 1000000, // Numeric version (e.g., 1000000 for 1.0.0)
    "versionName": "1.0.0", // Display version
    "icon": "$media:app_icon", // Application icon
    "label": "$string:app_name", // Application name (from string resources)
    "minAPIVersion": 9, // Minimum supported HarmonyOS API version
    "targetAPIVersion": 9 // Target HarmonyOS API version
  }
}
```

### 3.2 module.json5
This file configures module-specific settings (e.g., main entry, permissions):
```json
{
   "module": {
      "name": "entry",
      "type": "entry",
      "description": "$string:main_desc",
      "mainElement": "EntryAbility",
      "deviceTypes": [
         "phone",
         "tablet",
         "2in1"
      ],
      "srcEntry": "./ets/stage/AppStage.ets",
      "deliveryWithInstall": true,
      "installationFree": false,
      "pages": "$profile:main_pages",
      "routerMap": "$profile:route_map",
      "abilities": [
         {
            "name": "EntryAbility",
            "srcEntry": "./ets/ability/EntryAbility.ets",
            "description": "$string:entry_desc",
            "icon": "$media:app_icon",
            "label": "$string:ability_label",
            "startWindowIcon": "$media:app_icon",
            "startWindowBackground": "$color:start_window_background",
            "exported": true,
            "skills": [
               {
                  "entities": [
                     "entity.system.home"
                  ],
                  "actions": [
                     "action.system.home"
                  ]
               }
            ],
            "backgroundModes": [
               // 长时任务类型的配置项
               "audioRecording",
               "audioPlayback"
            ]
         }
      ],
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
   }
}
```

### 3.3 agconnect-services.json
Download this configuration file from AGC and place it in the project root directory:
1. In AGC, go to **Project Settings > General Information**.
2. Under **App Information**, click **agconnect-services.json** to download the file.
3. Copy the file to your DevEco Studio project's root directory.
This file contains AGC service configurations and must be included in the build.

## 4. Filling in Application Information for Release

Before submitting your application for review, complete the following information in AGC:

### 4.1 Basic Information
- **Application Name**: Official name displayed in the HarmonyOS Gallery.
- **Application Icon**: 216x216px PNG with transparent background.
- **Application Screenshots**: 3-5 screenshots of the application on different devices.
- **Description**: Detailed app introduction (supports markdown formatting).
- **What's New**: Changes in the current version.

### 4.2 Categorization and Pricing
- **Category**: Select the most appropriate category (e.g., Tools, Entertainment).
- **Subcategory**: Refine the application's classification.
- **Pricing**: Choose free or paid, and set price tiers if applicable.
- **Age Rating**: Select the appropriate age group (e.g., 3+, 12+).

### 4.3 Contact Information
- **Developer Email**: For communication regarding application review.
- **Developer Phone**: Alternative contact method for urgent issues.
- **Privacy Policy URL**: Required for applications handling user data.

## 5. Linking DevEco Studio Project with AGC

To enable AGC services in your application:
1. In DevEco Studio, open your project and go to **File > Project Structure**.
2. Select **Project Settings > AppGallery Connect**.
3. Click **Sign In** and log in with your Huawei Developer Account.
4. Select the project and application created in AGC.
5. Click **Apply** to link the project.
6. Sync project dependencies by clicking **Sync Now** in the notification bar.

## 6. Common Issues and Solutions

### 6.1 Package Name Mismatch
- **Issue**: Error during build: "Bundle name does not match AGC configuration".
- **Solution**: Ensure `bundleName` in `app.json5` exactly matches the package name in AGC application settings.

### 6.2 Missing agconnect-services.json
- **Issue**: Service initialization failed with "agconnect-services.json not found".
- **Solution**: Download the file from AGC and place it in the project root directory; verify file permissions.

### 6.3 Unsupported Device Types
- **Issue**: Application rejected with "Device type not supported".
- **Solution**: Update `deviceTypes` in `module.json5` to match the devices selected in AGC.

### 6.4 API Version Compatibility
- **Issue**: Build failed due to "API version not supported".
- **Solution**: Ensure `minAPIVersion` and `targetAPIVersion` in `app.json5` match the HarmonyOS SDK version installed in DevEco Studio.

## 7. Official Resources

- [AppGallery Connect Application Creation Guide](https://developer.huawei.com/consumer/cn/doc/distribution/app/agc-create-app-0000001504663401)
- [HarmonyOS Project Configuration Documentation](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ide-project-config-0000001504663397)
- [agconnect-services.json Configuration Guide](https://developer.huawei.com/consumer/cn/doc/development/AppGallery-connect-Guides/agc-get-started-android-0000001058283685)
- [Huawei Developer Alliance Help Center](https://developer.huawei.com/consumer/cn/forum/block/agc)

## 8. Summary

Creating and configuring your application in AppGallery Connect is a foundational step in the HarmonyOS Next release process. By carefully following the steps to create a project, configure core files (`app.json5`, `module.json5`, `agconnect-services.json`), and complete application information, developers can ensure seamless integration with AGC services and compliance with release requirements. Always verify configurations match between DevEco Studio and AGC to avoid common pitfalls during build and review.