# Application Compliance and Privacy Policy Explained

During the HarmonyOS Next app publishing process, compliance and privacy policy are top priorities for review. This article details compliance requirements, privacy policy writing tips, correct dynamic permission requests, common issues, and official resources to help developers pass the review smoothly.

## 1. Application Compliance Requirements

1. [**User Agreement and Privacy Policy**](https://developer.huawei.com/consumer/cn/doc/app/50129-07)
   - You must write a user agreement and privacy policy for your app, complying with the Personal Information Protection Law, Cybersecurity Law, and other relevant regulations.
   - The privacy policy must be a standalone page, easily accessible within the app (e.g., settings, first launch pop-up).
2. **Dynamic Permission Requests**
   - Only request permissions when necessary for a feature.
   - Permission pop-ups must clearly state the purpose and usage scenario, e.g., "To enable the camera feature, we need access to your camera permission."
   - Do not request all permissions at once; avoid "bundled authorization."
3. **Payment and Data Security**
   - Apps involving payments must integrate Huawei Pay SDK and pass PCI DSS and other security certifications.
   - User data must be encrypted in storage and transmission; do not collect irrelevant personal information.
4. **Third-party SDK Compliance**
   - Check all integrated third-party SDKs for compliance and no malicious behavior, and disclose SDK usage in the privacy policy.
   - When integrating third-party logins (e.g., WeChat), you must first integrate Huawei login.

## 2. Privacy Policy Writing Tips

- Clearly list what user information is collected, its purpose, storage method, and third-party sharing.
- Explain how users can withdraw consent, delete data, or contact support.
- Provide a privacy policy update mechanism and effective date.
- Refer to the official template and adjust based on your business needs.

> **Tip**: Refer to the [Huawei Developer Alliance Privacy Policy Template](https://developer.huawei.com/consumer/cn/doc/development/AppGallery-connect-Guides/agcprivacy-0000001053628147) for writing.

## 3. Dynamic Permission Request Practices

- Request permissions via pop-up only when the user first uses the relevant feature; for non-essential permissions, denial should not affect core functionality.
- Permission explanations must be specific and easy to understand; avoid vague statements.
- Provide fallback or friendly prompts if permission is denied.
- Update the privacy policy promptly when permissions change.

## 4. Common Issues and Tips

- **Missing or Incomplete Privacy Policy**: This will likely cause review rejection.
- **Unclear Permission Explanations**: Always specify the purpose in pop-ups; avoid vague statements like "access all your information."
- **Undisclosed Third-party SDKs**: All SDK purposes and data flows must be explained in the privacy policy, including: usage purpose, scenario, collected permissions, and user info.

## 5. Official Reference Links

- [Huawei Developer Alliance Account Center](https://developer.huawei.com/consumer/cn/)
- [Privacy Policy Compliance Guide](https://developer.huawei.com/consumer/cn/doc/app/50128)
- [Review Policy](https://developer.huawei.com/consumer/cn/doc/app/50000)

## 6. Summary

Application compliance and privacy policy are core to HarmonyOS Next app review. Prepare compliance materials in advance, strictly follow official requirements for privacy policy display, and ensure dynamic permission requests are compliant and transparent. If in doubt, consult official docs or the community to ensure your app passes review and gains user trust. 