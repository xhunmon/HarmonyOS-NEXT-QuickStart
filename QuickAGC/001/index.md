# Complete Guide to Developer Qualification and Account Registration

Before officially developing and launching a HarmonyOS Next application, developers must first obtain legal developer qualifications and complete the registration and real-name authentication of a Huawei Developer account. This article will detail the registration process, precautions, common issues, and useful resources to help you successfully take the first step on the path to launching your application.

## 1. Developer Account Registration and Qualification Authentication

### AGC Project Configuration File (Huawei Login)
After creating the application, obtain the configuration parameters from the AGC console: https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/account-client-id
```json
"module": {
  "name": "<name>",
  "type": "entry",
  "description": "<description>",
  "mainElement": "<mainElement>",
  "deviceTypes": [],
  "pages": "<pages>",
  "abilities": [],
  "metadata": [ // Configuration information is as follows
    {
      "name": "client_id",
      "value": "<Client ID obtained in the previous step>"
    }
  ]
 }
```

### Then you can integrate login
```json
//https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/account-phone-unionid-login
  getQuickLoginAnonymousPhone() {
    // Create authorization request and set parameters
    const authRequest = new authentication.HuaweiIDProvider().createAuthorizationWithHuaweiIDRequest();
    // To obtain an anonymous phone number, you need to pass the scope "quickLoginAnonymousPhone". You need to apply for the "Huawei Account One-Click Login" permission before passing the parameter, otherwise error code 1001502014 will be returned
    authRequest.scopes = ['quickLoginAnonymousPhone'];
    // Used to prevent cross-site request forgery
    authRequest.state = util.generateRandomUUID();
    // This parameter must be set to false for one-click login scenarios
    authRequest.forceAuthorization = false;
    const controller = new authentication.AuthenticationController();
    try {
      controller.executeRequest(authRequest).then((response: authentication.AuthorizationWithHuaweiIDResponse) => {
        // Obtained anonymous phone number
        const anonymousPhone = response.data?.extraInfo?.quickLoginAnonymousPhone as string;
        if (anonymousPhone) {
          hilog.info(0x0000, 'testTag', 'Succeeded in authentication.');
          const quickLoginAnonymousPhone: string = anonymousPhone;
          return;
        }
        hilog.info(0x0000, 'testTag', 'Succeeded in authentication. AnonymousPhone is empty.');
        // Anonymous phone number not obtained, the application needs to jump to other login pages
      }).catch((error: BusinessError) => {
        this.dealAllError(error);
      })
    } catch (error) {
      this.dealAllError(error);
    }
  }


  // Error handling
  dealAllError(error: BusinessError): void {
    hilog.error(0x0000, 'testTag',
      `Failed to get quickLoginAnonymousPhone, errorCode is ${error.code}, errorMessage is ${error.message}`);
    // In scenarios where app login involves UI interaction, it is recommended to prompt users according to the following error code guidelines
    if (error.code === ErrorCode.ERROR_CODE_LOGIN_OUT) {
      // Huawei account not logged in. The application needs to display other login methods
    } else if (error.code === ErrorCode.ERROR_CODE_NETWORK_ERROR) {
      // Network exception, please check current network status and try again or display other login methods
    } else if (error.code === ErrorCode.ERROR_CODE_INTERNAL_ERROR) {
      // Login failed, the application needs to display other login methods
    } else if (error.code === ErrorCode.ERROR_CODE_USER_CANCEL) {
      // User canceled authorization
    } else if (error.code === ErrorCode.ERROR_CODE_SYSTEM_SERVICE) {
      // System service exception, the application needs to display other login methods
    } else if (error.code === ErrorCode.ERROR_CODE_REQUEST_REFUSE) {
      // Duplicate request, no need for application to handle
    } else {
      // Application login failed, the application needs to display other login methods
    }
  }


  export enum ErrorCode {
    // Account not logged in
    ERROR_CODE_LOGIN_OUT = 1001502001,
    // Network error
    ERROR_CODE_NETWORK_ERROR = 1001502005,
    // Internal error
    ERROR_CODE_INTERNAL_ERROR = 1001502009,
    // User canceled authorization
    ERROR_CODE_USER_CANCEL = 1001502012,
    // System service exception
    ERROR_CODE_SYSTEM_SERVICE = 12300001,
    // Duplicate request
    ERROR_CODE_REQUEST_REFUSE = 1001500002
  }
  ...
```

Developer qualification refers to the legal qualification for developing and publishing applications obtained by developers after registering on the official Huawei platform and passing real-name authentication. Both individuals and enterprises must have developer qualifications to publish and manage applications on the [AppGallery Connect (AGC)](https://developer.huawei.com/consumer/cn/service/josp/agc/index.html#/) platform.

## 2. Account Type Selection

Huawei Developer accounts are divided into two types: "Individual Account" and "Enterprise Account":
- **Individual Account**: Suitable for individual developers, with a simple registration process, ideal for small applications or personal projects.
- **Enterprise Account**: Suitable for companies and organizations, supporting multi-person collaboration and enterprise-level services, requiring business license and other materials.

> **Recommendation**: If conditions permit, prioritize registering an enterprise account, which will facilitate subsequent qualification review, ad integration, and application promotion.

## 3. Detailed Registration Process

1. **Visit Huawei Developer Alliance Official Website**
   [Huawei Developer Registration Portal](https://developer.huawei.com/consumer/cn/)

2. **Click "Register" and Fill in Basic Information**
   You need to prepare a frequently used email, phone number, and set a secure password. Many functions also require real-name authentication and bank card binding.

3. **Verification and Login**
   Follow the system prompts to complete registration and login.

4. **Real-name Authentication**
   - Individual Account: Upload photos of the front and back of your ID card, and fill in your real name and ID number.
   - Enterprise Account: Upload business license, legal representative's ID card, and enterprise information.

5. **Wait for Review**
   Real-name authentication is usually completed within 1-3 working days. After approval, you can use all developer functions.

## 4. Common Issues and Suggestions to Avoid Pitfalls

- **Inconsistent Information**: Ensure that the registration information is completely consistent with the document information, otherwise the review will fail.
- **Blurry or Non-compliant Photos**: Uploaded document photos must be clear, unobstructed, and free from reflections and watermarks.
- **Changes in Enterprise Legal Person Information**: If the enterprise legal person changes, update the relevant information in the background and re-authenticate in a timely manner.
- **Unable to Receive Verification Email**: Check the spam folder or change to a frequently used email.

## 5. Know Where to Find Information When Encountering Problems

- [Official Documentation Search](https://developer.huawei.com/consumer/cn/doc/search?type=all&val=%E4%B8%8A%E6%9E%B6&versionValue=all)
- [HarmonyOS Next Forum](https://developer.huawei.com/consumer/cn/forum/block/harmonyos-next)

## 6. Summary

Developer qualification and account registration are the first steps in launching a HarmonyOS Next application. It is recommended to prepare all materials in advance, strictly follow the official process, and consult official documents or post for help on the forum when encountering problems. After successful registration, you can officially start the development and launch journey of your HarmonyOS Next application!