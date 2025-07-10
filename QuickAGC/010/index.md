# HarmonyOS Next Application Review and Release Process


After uploading the package and qualification materials, HarmonyOS Next applications enter the review and publication phase on AppGallery Connect (AGC). Only after passing the review can the application be officially launched. This article details the review process, publication considerations, common rejection reasons, troubleshooting strategies, and official resources to help developers complete the final step of publication.

## 1. Review Process Overview

1. **Submit for Review**
  - Confirm all information is correct on AGC and click "Submit for Review".

2. **Preliminary Automated Checks**
  - The system automatically verifies package signature, name, permissions, and material compliance.
  - If checks pass, the application proceeds to manual review.

3. **Manual Review**
  - Reviewers comprehensively examine functionality, UI, compliance, privacy policy, etc.
  - Focus areas: user agreements, privacy policies, permission requests, and functional stability.

4. **Review Result Notification**
  - **Approved**: The application enters the publication state and launches on AppGallery.
  - **Rejected**: The platform provides rejection reasons; modifications must be made before resubmission.

## 2. Publication Considerations

- **Review Cycle**: Typically 3-7 business days; may extend for complex applications or supplementary materials.
- **Gray Release**: Use gray release to launch to a small user group first, verifying compatibility and feedback.
- **Multi-Device Support**: If adapting to multiple devices, test each type and declare supported devices.
- **Subsequent Updates**: Each version update requires resubmission; plan testing and material preparation in advance.

## 3. Common Rejection Reasons

- **Missing or Inconsistent Privacy Policy**: Not uploaded or content mismatches actual functionality.
- **Unclear Permission Descriptions**: Permission pop-ups lack clear purpose explanations.
```typescript
// Check this file and verify if AGC requests additional dangerous permissions
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
      "abilities": ["EntryAbility"],
      "when": "always"
    }
  },
  {
    "name": "ohos.permission.MICROPHONE",
    "reason": "$string:permission_audio_record",
    "usedScene": {
      "abilities": ["EntryAbility"],
      "when": "inuse"
    }
  }
]
```  

- **Crashes, Lag, or Functional Issues**: Unstable core functionality.
- **Incomplete or Unclear Materials**: Non-compliant software copyright certificates, privacy policies, screenshots, etc.
- **Inconsistent Package Name/Signature/Version**: Mismatch with AGC information.
```typescript
// Configure application information in app.json5:
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

## 4. Troubleshooting Strategies

- **Review Rejection Reasons**: Address each feedback item systematically.
- **Supplement Materials**: Ensure all materials are authentic, clear, and compliant.
- **Optimize Functionality**: Fix crashes, lag, and stability issues.
- **Multi-Device Testing**: Verify functionality on all declared supported devices.
- **Proactive Communication**: Use AGC support tickets or developer communities for complex issues.
- **Leverage Developer Assistants**: If rejection details are unclear, consult HarmonyOS developer assistants for feedback.

## 5. Official References

- AppGallery Connect Review & Publication Guide
- HarmonyOS Next Documentation
- Huawei Developer Alliance Account Center

## 6. Conclusion

Review and publication are the final steps for launching HarmonyOS Next applications. Developers should:
1. Strictly adhere to official requirements for materials and functionality.
2. Promptly address rejection feedback and optimize product quality.
3. Utilize gray release and multi-device testing strategies.
4. Maintain proactive communication with review teams.  
   This ensures smooth application launch and user recognition.