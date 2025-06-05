# Complete Guide to Domain Registration and Network Compliance

During the HarmonyOS Next app publishing process, if your app involves network services (such as APIs, cloud storage, user registration/login, etc.), you must complete domain registration and ICP filing. This article details the domain registration and compliance process, common issues, and official resources to help developers pass network compliance review smoothly.

## 1. When Is Domain Filing Required?

- Any app involving network services (such as server access, cloud APIs, push notifications, data sync, etc.) must use a registered and filed domain.
- Standalone local apps (completely offline) do not require filing.
- Domain filing is a legal requirement in mainland China; unfiled domains cannot be used in apps.

## 2. Domain Registration Process

1. Choose a domain registrar (e.g., Huawei Cloud, Alibaba Cloud, Tencent Cloud, etc.).
2. Check and purchase a suitable domain; choose a short domain related to your app brand.
3. Complete real-name authentication, submit the registration application, and pay the fee.
4. After successful registration, log in to the registrar's backend to manage the domain.

> **Tip**: Prefer major mainland China cloud providers for easier filing and better support.

## 3. ICP Filing Process

1. Log in to the registrar's backend (e.g., Huawei Cloud), go to the "ICP Filing" page.
2. Fill in the filing subject (enterprise/individual), website info, responsible person info, etc.
3. Upload required documents (e.g., business license, responsible person's ID, website commitment letter, etc.).
4. Wait for the registrar's preliminary review; after passing, submit to the authority for final review.
5. Once approved, obtain the filing number and display it at the bottom of your app and website.

> **Notes**:
> - Enterprises need to provide a business license; individuals need an ID card.
> - Filing usually takes 7-20 business days; plan ahead.
> - During filing, the domain cannot be used for production; use a test domain or IP for debugging.

## 4. Common Network Compliance Issues

- **Unfiled Domain Blocked**: Unfiled domains will be detected and cause review rejection.
- **Inconsistent Filing Info**: Filing subject must match app developer info, or it may be rejected.
- **Filing Number Not Displayed**: After approval, display the filing number prominently in the app and website.
- **Incomplete or Unclear Documents**: Upload clear, authentic documents to avoid delays.

## 5. Official Reference Links

- [Huawei Cloud ICP Filing Service](https://support.huaweicloud.com/icp/)
- [MIIT ICP Filing System](https://beian.miit.gov.cn/)
- [Huawei Developer Alliance Account Center](https://developer.huawei.com/consumer/cn/)
- [HarmonyOS Next Official Docs](https://developer.huawei.com/consumer/cn/doc/)

## 6. Summary

Domain registration and network compliance are essential for HarmonyOS Next app publishing. Register your domain and complete filing in advance to ensure all network services are compliant. If you encounter issues, consult the registrar or MIIT docs, or seek help in the developer community to ensure smooth review and secure launch. 