# Application Compliance and Privacy Policy Explained

Compliance and privacy policies are critical during the HarmonyOS Next app review process. This article details compliance requirements, key points for privacy policy drafting, best practices for dynamic permission requests, common issues, and official references to help developers pass review smoothly.

## 1. App Compliance Requirements

1. **User Agreement and Privacy Policy**
   - Must create user agreements and privacy policies compliant with laws like the *Personal Information Protection Law* and *Cybersecurity Law*.
   - Privacy policies should be standalone pages with easy access (e.g., settings page, first-launch popup).

   Implementation example for privacy policy popup:
```typescript
import { IntentUtil, Log } from 'common_utils';
import { loginComponentManager } from '@kit.AccountKit';
import { curves } from '@kit.ArkUI';
import { ResColor } from '../res/ResColor';

const TAG = 'PrivacyCheckbox';

@ComponentV2
export struct PrivacyCheckbox {
  // Agreement content
  @Param texts: loginComponentManager.PrivacyText[] = [];
  // Agreement checkbox status
  @Param isCheckboxSelected: boolean = false;
  @Event boxChange: (select: boolean) => void;
  // Triggers unselected animation
  @Param unselectAnimalCount: number = 0;
  // Custom click handling
  @Event onCustomItemClick?: (item: loginComponentManager.PrivacyText) => void;
  // Animation
  @Local translateX: number = 0;

  build() {
    Row() {
      Row() {
        Checkbox({ name: 'privacyCheckbox', group: 'privacyCheckboxGroup' })
          .width(24)
          .height(24)
          .focusable(true)
          .focusOnTouch(true)
          .selectedColor(ResColor.themeLight())
          .margin({ top: 0 })
          .select(this.isCheckboxSelected)
          .unselectedColor(this.translateX === 0 ? Color.Gray : Color.Red)
          .translate({ x: this.translateX })
          .animation({ curve: curves.springMotion() })
          .onChange((value: boolean) => {
            this.boxChange(value);
            Log.d(TAG, () => `agreementChecked: ${value}`);
          })
      }

      Row() {
        Text() {
          ForEach(this.texts, (item: loginComponentManager.PrivacyText, _index: number) => {
            if (item?.type == loginComponentManager.TextType.PLAIN_TEXT && item?.text) {
              Span(item?.text)
                .fontColor(ResColor.desc())
                .fontWeight(FontWeight.Regular)
                .fontSize($r('sys.float.ohos_id_text_size_body3'))
            } else if (item?.type == loginComponentManager.TextType.RICH_TEXT && item?.text) {
              Span(item?.text)
                .fontColor(ResColor.themeLight())
                .fontWeight(FontWeight.Medium)
                .fontSize($r('sys.float.ohos_id_text_size_body3'))
                .focusable(true)
                .focusOnTouch(true)
                .onClick(() => {
                  Log.d(TAG, () => `click privacy text tag: ${item.tag}`);
                  if (this.onCustomItemClick !== undefined) {
                    this.onCustomItemClick(item);
                  } else if (item.tag) { // Implement page navigation based on item.tag
                    IntentUtil.jumpUrl(item.tag)
                  }
                })
            }
          })
        }
      }
      .layoutWeight(1)
      .margin({ left: 12 })
      .constraintSize({ minHeight: 24 })
    }
    .width('100%')
    .alignItems(VerticalAlign.Top)
    .justifyContent(FlexAlign.Start)
  }

  @Monitor('unselectAnimalCount')
  checkAgreeAnimate() {
    if (this.isCheckboxSelected) {
      return;
    }
    let count = 10;
    let t = 0;
    const id = setInterval(() => {
      count -= 1;
      if (count === 0) {
        clearInterval(id);
        this.translateX = 1;
        return;
      }
      t += 30;
      this.translateX = 30 * Math.sin(t);
    }, 100)
  }
}
```

2. **Dynamic Permission Requests**
   - Request permissions only when functionality requires them.
   - Clearly explain usage purposes (e.g., "Camera access needed for photo features").

```typescript
// Request camera permission
async function requestCameraPermission() {
  let atManager = abilityAccessCtrl.createAtManager();
  try {
    let grantStatus = await atManager.checkAccessToken(abilityAccessCtrl.AtManager.PERMISSION_CAMERA);
    if (grantStatus === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED) {
      return true;
    }
    // Request permission
    let requestResult = await atManager.requestPermissionsFromUser(this.context, [abilityAccessCtrl.AtManager.PERMISSION_CAMERA]);
    if (requestResult.authResults === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED) {
      promptAction.showToast({ message: 'Camera permission granted' });
      return true;
    } else {
      promptAction.showToast({ message: 'Camera permission denied' });
      return false;
    }
  } catch (err) {
    console.error(`Permission request failed: ${JSON.stringify(err)}`);
    return false;
  }
}
```
- Avoid "bundled authorization" (requesting all permissions at once).

3. **Payment and Data Security**
   - Payment features must integrate Huawei Pay SDK with PCI DSS certification.
   - Encrypt user data during storage/transmission; prohibit collecting irrelevant personal information.

4. **Third-Party SDK Compliance**
   - Ensure integrated SDKs are compliant and disclosed in privacy policies.
   - When integrating third-party logins (e.g., WeChat), Huawei Login must be implemented first.

## 2. Privacy Policy Essentials

- Explicitly list collected data types, purposes, storage methods, and third-party sharing.
- Describe user rights: consent withdrawal, data deletion, customer support.
- Provide update mechanisms and effective dates.
- Adapt official templates to your business needs.


## 3. Dynamic Permission Best Practices

- Request permissions during first use of related features.
- Provide clear, understandable permission explanations.
- Implement fallback solutions for denied permissions.

```typescript
// Fallback handling for denied permissions
Button('Take Photo')
  .onClick(async () => {
    let hasPermission = await requestCameraPermission();
    if (hasPermission) {
      openCamera();
    } else {
      // Fallback: Open photo album
      promptAction.showToast({ message: 'Opening photo album instead' });
      setTimeout(() => openAlbum(), 1000);
    }
  })
```
- Update privacy policies when permission requirements change.

## 4. Common Issues and Solutions

- **Missing/incomplete privacy policies**: Primary cause for rejection.
- **Vague permission descriptions**: Avoid ambiguous phrases like "access all your information".
- **Undisclosed third-party SDKs**: Disclose all SDK purposes, data flows, and collected information.

## 5. Official References

- Huawei Developer Alliance
- Privacy Compliance Guide
- Review Policies

## 6. Conclusion

Compliance and privacy policies are fundamental to HarmonyOS Next app reviews. Prepare materials in advance, strictly follow official requirements, and ensure transparent permission handling. Consult official documentation or community forums when in doubt to ensure smooth approval and user trust.
 