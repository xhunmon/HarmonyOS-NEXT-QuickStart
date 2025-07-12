# Complete Guide to Obtaining and Configuring Signing Files

Signature files are critical for ensuring the security and legitimacy of HarmonyOS Next applications. They serve as digital credentials that verify the developer's identity and ensure the application has not been tampered with during distribution. This guide details the purpose of signature files, the application process via AppGallery Connect, configuration steps in project files, and best practices for management.

## 1. Understanding Signature File Types

HarmonyOS Next applications require two essential signature files during the release process:

### 1.1 Developer Certificate (.p12)
- A digital certificate issued to developers by Huawei, used to sign applications and verify the developer's identity.
- Contains the developer's public key and is encrypted with a password for security.
- Valid for a specific period (typically 1-3 years); must be renewed before expiration.

### 1.2 Profile File (.p7b)
- Binds the application's package name, developer certificate, and allowed installation devices (for debugging) or release channels (for production).
- Ensures the application can only be installed on authorized devices or distributed through approved channels.
- Must be regenerated when application information (package name, certificate, etc.) changes.

## 2. Applying for Signature Files via AppGallery Connect

### 2.1 Prerequisites
- A registered Huawei Developer account (individual or enterprise).
- A created HarmonyOS application project in AppGallery Connect.
- Completed identity verification (required for enterprise accounts).

### 2.2 Step-by-Step Application Process

1. **Log in to AppGallery Connect**
   Visit [AppGallery Connect](https://developer.huawei.com/consumer/cn/service/josp/agc/index.html#/myApp) and log in with your Huawei Developer account.

2. **Create or Select an Application**
   - If no project exists: Click "My Projects" > "New Project" > Follow the wizard to create a HarmonyOS application.
   - If project exists: Select the target application from the project list.

3. **Apply for a Developer Certificate**
   - Navigate to "Project Settings" > "General Information" > "Signing Certificates".
   - Click "Generate Certificate" and select certificate type (debug or release).
   - Fill in certificate information (password, alias, etc.) and download the .p12 file.
   - **Important**: Store the password and .p12 file securely; they cannot be retrieved if lost.

4. **Generate a Profile File**
   - In the same "Signing Certificates" section, click "Generate Profile".
   - Select the previously created certificate, specify the application bundle name, and choose distribution type (debug, release, or HarmonyOS Gallery).
   - For debug profiles: Add device UDIDs to restrict installation to specific test devices.
   - Download the generated .p7b Profile file.

## 3. Configuring Signature Files in the Project

### 3.1 Project Configuration Files
Signature information must be configured in two key files:

#### 3.1.1 build-profile.json5
This file stores build and signing configurations. Add or update the `signingConfigs` section:

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

#### 3.1.2 app.json5
Configure basic application information that must match the Profile file:

```json
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

### 3.2 Configuration in DevEco Studio

1. Open your project in DevEco Studio.
2. Go to "File" > "Project Structure" > "Project Settings" > "Signing Configs".
3. Click "+" to add a new signing configuration.
4. Fill in the following fields:
   - **Name**: default (or custom name)
   - **Signing Type**: HarmonyOS
   - **Store File**: Select the downloaded .p12 file
   - **Store Password**: Enter the certificate password
   - **Key Alias**: Enter the certificate alias
   - **Key Password**: Enter the key password
   - **Profile File**: Select the downloaded .p7b file
5. Click "OK" to save the configuration.

## 4. Common Issues and Troubleshooting

### 4.1 Certificate or Profile Mismatch
- **Symptom**: Packaging fails with "signature verification failed" or "profile mismatch".
- **Solution**: Ensure the bundle name in `app.json5` matches the one in the Profile file; verify the certificate and Profile are from the same AppGallery Connect project.

### 4.2 Lost Certificate Password
- **Symptom**: Unable to import the .p12 file due to incorrect password.
- **Solution**: There is no password recovery mechanism. Generate a new certificate and Profile file in AppGallery Connect.

### 4.3 Expired Profile File
- **Symptom**: Application installation fails with "profile expired".
- **Solution**: Regenerate the Profile file in AppGallery Connect and update the project configuration.

### 4.4 Debug Profile Not Working on Test Devices
- **Symptom**: Installed application crashes immediately or shows "unauthorized device".
- **Solution**: Ensure the test device's UDID is added to the debug Profile; regenerate the Profile if new devices are added.

## 5. Best Practices for Signature File Management

- **Secure Storage**: Store .p12 files and passwords in encrypted storage or password managers; never commit them to version control.
- **Regular Backups**: Create backups of certificates and Profile files and store them in multiple secure locations.
- **Certificate Renewal**: Set reminders for certificate expiration dates (typically 90 days before expiry) to avoid service interruptions.
- **Separate Environments**: Use different certificates and profiles for development, testing, and production environments to prevent accidental release of test versions.

## 6. Official Resources

- [AppGallery Connect Certificate Management Guide](https://developer.huawei.com/consumer/cn/doc/distribution/app/agc-certificate-management)
- [HarmonyOS Signature File Configuration Documentation](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ide-signing)
- [Huawei Developer Alliance Help Center](https://developer.huawei.com/consumer/cn/forum/block/agc)
- [DevEco Studio Signing Configuration Tutorial](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ide-project-config-0000001504663397)

## 7. Summary

Obtaining and configuring signature files correctly is a prerequisite for successfully releasing HarmonyOS Next applications. By following the application process in AppGallery Connect, properly configuring project files, and adhering to security best practices, developers can ensure their applications are secure, tamper-proof, and compliant with Huawei's release requirements. Always refer to official documentation for the latest updates on signature management processes.