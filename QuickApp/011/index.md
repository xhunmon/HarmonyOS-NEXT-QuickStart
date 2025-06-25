# ZeroOneApp Network Request Framework Design and Implementation

## Introduction

In modern application development, the network module serves as the core hub connecting the frontend and backend services. As an open-source practice project in the QuickApp series, ZeroOneApp's network module is built on the Axios library to create an efficient and scalable request framework. This article will deeply analyze the design philosophy and implementation details of the `NetRequest` class in ZeroOneApp, helping developers master best practices for network requests in HarmonyOS applications.

## Official Resources

- **Axios for HarmonyOS**: ZeroOneApp uses `@ohos/axios` as the basic network request library, which is a Promise-based HTTP client supporting request/response interception, data transformation, and other features [Axios Official Documentation](https://ohpm.openharmony.cn/#/cn/detail/@ohos%2Faxios)
- **HarmonyOS Network Development Guide**: Provides official specifications and recommendations for network requests, data security, etc. [HarmonyOS Network Development Documentation](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/http-request)

## Detailed Explanation

### 1. Core Design of Network Request Framework

The `NetRequest` class adopts a singleton pattern design to ensure there is only one network request instance in the application, avoiding resource waste from repeated creation. The core code structure is as follows:

```typescript:common/net/src/main/ets/core/NetRequest.ets
class NetRequest {
  private instance: AxiosInstance;
  private responseInterceptFunc?: (response: AxiosResponse) => AxiosResponse;
  private errorInterceptFunc?: (error: AxiosError) => AxiosError;

  constructor() {
    this.instance = axios.create({
      baseURL: API.base_url,
      timeout: 5000,
      connectTimeout: 60000,
      headers: {}
    });
    this.initIntercept(this.instance);
  }

  // Singleton instance export
  export const net: NetRequest = new NetRequest();
}
```

### 2. Implementation of Request and Response Interceptors

The framework implements unified request processing, response parsing, and error handling through interceptors:

```typescript
private initIntercept(instance: AxiosInstance) {
  // Response interceptor
  instance.interceptors.response.use((response: AxiosResponse) => {
    if (this.responseInterceptFunc) {
      return this.responseInterceptFunc(response);
    }
    return response;
  }, (error: AxiosError) => {
    // Error handling logic
    log.i(TAG, 'initIntercept error: code=' + error.code + ' | message=' + error.message);
    if (this.errorInterceptFunc) {
      return this.errorInterceptFunc(error);
    }
    return error;
  });
}
```

Advantages of the interceptor mechanism:
- Centralized processing of all requests/responses, reducing duplicate code
- Unified addition of authentication tokens, request headers, etc.
- Unified handling of error status codes such as 401 Unauthorized and 500 Server Error
- Support for custom interception logic extension

### 3. Request Method Encapsulation

The framework encapsulates Axios's POST method to provide a more concise API and unified callback handling:

```typescript
post<T, D>(url: string, data?: D, callback?: Callback<T>) {
  log.i(TAG, `[Request][${url}]: ${StringUtil.toStr(data)}`);
  this.instance
    .post<string, AxiosResponse<MagicResp<T>>, D>(url, data)
    .then((response: AxiosResponse<MagicResp<T>>) => {
      log.i(TAG, `[Response][${url}]: ${StringUtil.toStr(response.data)}`);
      // Response handling logic
    })
    .catch((error: Error) => {
      // Error handling logic
    })
    .finally(() => {
      if (callback && callback.onFinally) {
        callback.onFinally();
      }
    });
}
```

Features of the encapsulated POST method:
- Generic support for type safety
- Unified log printing for debugging
- Standardized callback interface (onSuccess/onFail/onFinally)
- Deep integration with MagicResp response model

## Practical Recommendations

### 1. Interceptor Usage Tips

```typescript
// Configure global response interception during application initialization
net.onResponseIntercept((response) => {
  // Unified response data processing
  if (response.data.code === 401) {
    // Handle unauthorized access, such as redirecting to login page
  }
  return response;
});
```

### 2. Dynamic Request Header Addition

```typescript
// Add authentication token after login
net.addHeader({
  'Authorization': `Bearer ${token}`,
  'X-Platform': 'HarmonyOS'
});
```

### 3. Error Handling Best Practices

```typescript
net.post<LoginResp, LoginReq>(API.login, {username, password}, {
  onSuccess: (data) => {
    // Handle success logic
  },
  onFail: (code, msg) => {
    if (code === -1) {
      promptAction.showToast({ message: 'Network connection failed' });
    } else {
      promptAction.showToast({ message: msg });
    }
  },
  onFinally: () => {
    //收尾操作，如隐藏加载动画
  }
});
```

## Summary and Outlook

ZeroOneApp's network request framework achieves code reuse, unified management, and flexible expansion through elegant design. Core advantages include:

1. **Reasonable Architecture Design**: Singleton pattern ensures efficient resource utilization, and interceptor mechanism achieves separation of cross-cutting concerns
2. **User-Friendly API Design**: Generic support and callback interface provide a good development experience
3. **Strong Extensibility**: Supports custom interception logic and request configuration
4. **Comprehensive Error Handling**: Covers various exception scenarios including network errors and business errors

Future features to consider adding:
- Request retry mechanism
- Network caching strategy
- Request priority management
- Batch request processing

## References

- [@ohos/axios npm package](https://ohpm.openharmony.cn/#/cn/detail/@ohos%2Faxios)
- [HarmonyOS Application Development Documentation](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/http-request)
- [RESTful API Design Best Practices](https://restfulapi.net/)