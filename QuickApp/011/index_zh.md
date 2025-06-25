# ZeroOneApp网络请求框架设计与实现

## 引言

在现代应用开发中，网络模块是连接前端与后端服务的核心枢纽。ZeroOneApp作为QuickApp系列文章的开源实践项目，其网络模块基于Axios库构建了一套高效、可扩展的请求框架。本文将深入解析ZeroOneApp中`NetRequest`类的设计理念与实现细节，帮助开发者掌握HarmonyOS应用中网络请求的最佳实践。

## 官方资源介绍

- **Axios for HarmonyOS**：ZeroOneApp采用`@ohos/axios`作为网络请求基础库，这是一个基于Promise的HTTP客户端，支持拦截请求和响应、转换请求数据和响应数据等特性[Axios官方文档](https://ohpm.openharmony.cn/#/cn/detail/@ohos%2Faxios)
- **HarmonyOS网络开发指南**：提供了网络请求、数据安全等方面的官方规范和建议[HarmonyOS网络开发文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/http-request)

## 详细讲解

### 1. 网络请求框架核心设计

`NetRequest`类采用单例模式设计，确保应用中只有一个网络请求实例，避免重复创建带来的资源浪费。核心代码结构如下：

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

  // 单例实例导出
  export const net: NetRequest = new NetRequest();
}
```

### 2. 请求与响应拦截器实现

框架通过拦截器机制实现了请求统一处理、响应统一解析和错误统一处理：

```typescript
private initIntercept(instance: AxiosInstance) {
  // 响应拦截器
  instance.interceptors.response.use((response: AxiosResponse) => {
    if (this.responseInterceptFunc) {
      return this.responseInterceptFunc(response);
    }
    return response;
  }, (error: AxiosError) => {
    // 错误处理逻辑
    log.i(TAG, 'initIntercept error: code=' + error.code + ' | message=' + error.message);
    if (this.errorInterceptFunc) {
      return this.errorInterceptFunc(error);
    }
    return error;
  });
}
```

拦截器机制的优势在于：
- 集中处理所有请求/响应，减少重复代码
- 统一添加认证令牌、设置请求头等
- 统一处理错误状态码，如401未授权、500服务器错误等
- 支持自定义拦截逻辑扩展

### 3. 请求方法封装

框架对Axios的POST方法进行了封装，提供了更简洁的API和统一的回调处理：

```typescript
post<T, D>(url: string, data?: D, callback?: Callback<T>) {
  log.i(TAG, `[请求][${url}]: ${StringUtil.toStr(data)}`);
  this.instance
    .post<string, AxiosResponse<MagicResp<T>>, D>(url, data)
    .then((response: AxiosResponse<MagicResp<T>>) => {
      log.i(TAG, `[响应][${url}]: ${StringUtil.toStr(response.data)}`);
      // 响应处理逻辑
    })
    .catch((error: Error) => {
      // 错误处理逻辑
    })
    .finally(() => {
      if (callback && callback.onFinally) {
        callback.onFinally();
      }
    });
}
```

封装后的POST方法具有以下特点：
- 泛型支持，提供类型安全
- 统一日志打印，便于调试
- 标准化的回调接口（onSuccess/onFail/onFinally）
- 与MagicResp响应模型深度集成

## 实用建议

### 1. 拦截器使用技巧

```typescript
// 应用初始化时配置全局响应拦截
net.onResponseIntercept((response) => {
  // 统一处理响应数据
  if (response.data.code === 401) {
    // 处理未授权，如跳转登录页
  }
  return response;
});
```

### 2. 请求头动态添加

```typescript
// 登录后添加认证令牌
net.addHeader({
  'Authorization': `Bearer ${token}`,
  'X-Platform': 'HarmonyOS'
});
```

### 3. 错误处理最佳实践

```typescript
net.post<LoginResp, LoginReq>(API.login, {username, password}, {
  onSuccess: (data) => {
    // 处理成功逻辑
  },
  onFail: (code, msg) => {
    if (code === -1) {
      promptAction.showToast({ message: '网络连接失败' });
    } else {
      promptAction.showToast({ message: msg });
    }
  },
  onFinally: () => {
    // 隐藏加载动画等收尾操作
  }
});
```

## 总结与展望

ZeroOneApp的网络请求框架通过优雅的设计实现了代码复用、统一管理和灵活扩展。核心优势包括：

1. **架构设计合理**：单例模式确保资源高效利用，拦截器机制实现横切关注点分离
2. **API设计友好**：泛型支持和回调接口提供良好的开发体验
3. **可扩展性强**：支持自定义拦截逻辑和请求配置
4. **错误处理完善**：覆盖网络错误、业务错误等多种异常场景

未来可以考虑添加的功能：
- 请求重试机制
- 网络缓存策略
- 请求优先级管理
- 批量请求处理

## 参考链接

- [@ohos/axios npm包](https://ohpm.openharmony.cn/#/cn/detail/@ohos%2Faxios)
- [HarmonyOS应用开发文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/http-request)
- [RESTful API设计最佳实践](https://restfulapi.net/)