# [应用合规性](https://developer.huawei.com/consumer/cn/doc/app/50104)与隐私政策全解

在HarmonyOS Next应用上架过程中，合规性和隐私政策是审核的重中之重。本文将详细介绍应用合规性要求、隐私政策编写要点、动态权限申请的正确做法，以及常见问题和官方参考资源，帮助开发者顺利通过审核。

## 一、应用合规性要求

1. [**用户协议与隐私政策**](https://developer.huawei.com/consumer/cn/doc/app/50129-07)
   - 必须为应用编写用户协议和隐私政策，内容需符合《个人信息保护法》《网络安全法》等相关法规。
   - 隐私政策需单独成页，且在应用内易于访问（如设置页、首次启动弹窗等）。

   以下是隐私政策弹窗实现的案例代码：

```typescript
import { IntentUtil, Log } from 'common_utils';
import { loginComponentManager } from '@kit.AccountKit';
import { curves } from '@kit.ArkUI';
import { ResColor } from '../res/ResColor';

const TAG = 'PrivacyCheckbox';

@ComponentV2
export struct PrivacyCheckbox {
  // 协议内容
  @Param texts: loginComponentManager.PrivacyText[] = [];
  // 是否勾选协议
  @Param isCheckboxSelected: boolean = false;
  @Event boxChange: (select: boolean) => void;
  //通过外面改变unselectAnimalCount的值来触发未选中的动画
  @Param unselectAnimalCount: number = 0;
  //如果实现了这个的话就不会自动跳转 tag
  @Event onCustomItemClick?: (item: loginComponentManager.PrivacyText) => void;
  //动画
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
                  } else if (item.tag) { // 应用需要根据item.tag实现协议页面的跳转逻辑。
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


2. **动态权限申请**
   - 仅在确需使用相关功能时，才弹窗申请权限。
   - 权限弹窗需明确说明用途和使用场景，例如"为了实现拍照功能，需要访问您的相机权限"。

```typescript
// 检查并申请相机权限
async function requestCameraPermission() {
  let atManager = abilityAccessCtrl.createAtManager();
  try {
    let grantStatus = await atManager.checkAccessToken(abilityAccessCtrl.AtManager.PERMISSION_CAMERA);
    if (grantStatus === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED) {
      // 权限已授予
      return true;
    }
    // 申请权限
    let requestResult = await atManager.requestPermissionsFromUser(this.context, [abilityAccessCtrl.AtManager.PERMISSION_CAMERA]);
    if (requestResult.authResults[0] === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED) {
      promptAction.showToast({ message: '相机权限申请成功' });
      return true;
    } else {
      promptAction.showToast({ message: '相机权限被拒绝，无法使用拍照功能' });
      return false;
    }
  } catch (err) {
    console.error(`权限申请失败: ${JSON.stringify(err)}`);
    return false;
  }
}
```
   - 不得一次性申请所有权限，避免"捆绑授权"。
3. **支付与数据安全**
   - 涉及支付功能的应用，必须接入华为支付SDK，并通过PCI DSS等安全认证。
   - 用户数据需加密存储和传输，严禁收集与业务无关的个人信息。
4. **第三方SDK合规**
   - 检查所有集成的第三方SDK，确保其合规、无恶意行为，并在隐私政策中披露SDK用途。
   - 集成第三方登录时（如微信），必须要先集成华为登录。

## 二、隐私政策编写要点

- 明确列出收集哪些用户信息、用途、存储方式、第三方共享情况。
- 说明用户如何撤回授权、删除数据、联系客服等权利。
- 提供隐私政策的更新机制和生效日期。
- 参考官方模板，结合自身业务实际进行调整。

> **实用建议**：可参考华为开发者联盟提供的[隐私政策模板](https://developer.huawei.com/consumer/cn/doc/development/AppGallery-connect-Guides/agcprivacy-0000001053628147)进行编写。

## 三、动态权限申请实践

- 在用户首次使用相关功能时弹窗申请权限，避免"上来就全要"；而且要注意非必要权限时，拒绝不能影响功能的使用。
- 权限说明要具体、易懂，避免模糊表述。
- 拒绝授权后应有降级方案或友好提示。

```typescript
// 权限拒绝后的降级处理示例
Button('拍照')
  .onClick(async () => {
    let hasPermission = await requestCameraPermission();
    if (hasPermission) {
      // 正常打开相机
      openCamera();
    } else {
      // 降级处理：显示相册选择
      promptAction.showToast({ message: '将为您打开相册选择图片' });
      setTimeout(() => openAlbum(), 1000);
    }
  })
```
- 权限变更时及时更新隐私政策。

## 四、常见问题与避坑指南

- **隐私政策缺失或内容不全**：极易导致审核驳回。
- **权限说明不清晰**：务必在弹窗中写明用途，避免"获取您的全部信息"等模糊表述。
- **第三方SDK未披露**：所有SDK用途和数据流向都需在隐私政策中说明，准确的说明包括：使用目的、使用场景、收集权限和用户信息。

## 五、官方参考链接

- [华为开发者联盟账号中心](https://developer.huawei.com/consumer/cn/)
- [隐私政策合规指引](https://developer.huawei.com/consumer/cn/doc/app/50128)
- [审核政策](https://developer.huawei.com/consumer/cn/doc/app/50000)

## 六、总结

应用合规性和隐私政策是HarmonyOS Next应用上架审核的核心环节。建议开发者提前准备好合规材料，严格按照官方要求编写和展示隐私政策，动态权限申请务必合规、透明。遇到疑问时，优先查阅官方文档或在社区发帖求助，确保应用顺利通过审核并赢得用户信任。
