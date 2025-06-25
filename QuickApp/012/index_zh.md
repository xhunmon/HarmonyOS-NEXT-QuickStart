# ZeroOneApp网络下载功能实现与优化

## 引言

在移动应用开发中，文件下载是一项常见且关键的功能，涉及网络请求、进度监听、文件操作和异常处理等多个方面。ZeroOneApp的网络模块不仅提供了基础的HTTP请求能力，还封装了一套高效可靠的下载功能，支持进度反馈、文件管理和错误恢复。本文将深入剖析`NetRequest`类中的下载实现细节，探讨其设计思路与优化策略，为HarmonyOS应用开发中的下载功能实现提供参考。

## 官方资源介绍

- **HarmonyOS文件管理**：提供了应用沙箱内文件操作的完整API，包括创建、删除、读写文件等功能[HarmonyOS文件管理文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/app-file-access)
- **Axios下载进度**：`@ohos/axios`库支持通过`onDownloadProgress`回调获取下载进度信息[Axios请求配置](https://ohpm.openharmony.cn/#/cn/detail/@ohos%2Faxios)
- **HarmonyOS网络状态管理**：提供网络连接状态监听能力，可用于优化下载体验[HarmonyOS网络状态](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/http-request)

## 详细讲解

### 1. 下载功能核心实现

ZeroOneApp的下载功能通过`download`方法实现，该方法封装了文件下载的完整生命周期，包括请求配置、进度监听、文件处理和结果回调：

```typescript:common/net/src/main/ets/core/NetRequest.ets
download(url: string, filePath: string, onResult: (status: number, progress: number, msg: string) => void) {
  log.i(TAG, `[下载][${url}]: ${filePath}`);
  FileUtil.delete(filePath);
  axios({
    url: url,
    method: 'get',
    context: context,
    filePath: filePath,
    onDownloadProgress: (progressEvent: AxiosProgressEvent): void => {
      if (progressEvent === undefined || progressEvent.total === undefined) {
        return;
      }
      const progress = progressEvent.total ? Math.ceil(progressEvent.loaded / progressEvent.total * 100) : 0;
      log.i(TAG, `progress: ${progress} | ${StringUtil.toStr(progressEvent)}`);
      onResult(2, progress, '下载中');
    }
  }).then((response: AxiosResponse<string>) => {
    log.i(TAG, `response: ${StringUtil.toStr(response)}`);
    if (response.status === 200) {
      onResult(1, 100, response.data);
    } else {
      FileUtil.delete(filePath);
      onResult(-1, 0, response.data);
    }
  }).catch((error: AxiosError) => {
    log.e(TAG, `[异常][${url}]: ${StringUtil.toStr(error)}`);
    FileUtil.delete(filePath);
    onResult(-1, 0, error.message);
  })
}
```

### 2. 关键技术点解析

#### 2.1 文件路径处理与冲突解决

下载前先删除目标路径文件，避免旧文件干扰和路径冲突：
```typescript
FileUtil.delete(filePath);
```
这一操作确保了每次下载都是全新开始，防止因文件残留导致的校验错误或版本混乱。

#### 2.2 进度监听实现

通过Axios的`onDownloadProgress`回调实时获取下载进度：
```typescript
onDownloadProgress: (progressEvent: AxiosProgressEvent): void => {
  if (progressEvent === undefined || progressEvent.total === undefined) {
    return;
  }
  const progress = progressEvent.total ? Math.ceil(progressEvent.loaded / progressEvent.total * 100) : 0;
  onResult(2, progress, '下载中');
}
```
进度计算采用`loaded/total*100`公式，并使用`Math.ceil`确保进度值为整数百分比，便于UI展示。

#### 2.3 状态码设计

下载状态通过回调函数的`status`参数传递，定义了四种清晰的状态：
- `-1`：下载失败
- `0`：未下载（初始状态）
- `1`：下载成功
- `2`：正在下载
这种状态设计使调用方能够方便地处理不同下载阶段的UI展示和业务逻辑。

#### 2.4 错误处理与资源清理

下载失败时，框架会自动删除不完整文件，避免占用存储空间：
```typescript
.catch((error: AxiosError) => {
  log.e(TAG, `[异常][${url}]: ${StringUtil.toStr(error)}`);
  FileUtil.delete(filePath);
  onResult(-1, 0, error.message);
})
```
这一机制保证了文件系统的清洁性，防止无效文件堆积。

## 实用建议

### 1. 下载功能完整使用示例

```typescript
// 下载APK更新包
net.download(
  'https://example.com/xxx.zip',
  getContext().filesDir + '/update.apk',
  (status, progress, msg) => {
    switch (status) {
      case 2:
        // 更新进度条
        this.downloadProgress = progress;
        break;
      case 1:
        // 下载成功，安装APK
        this.installApk(getContext().filesDir + '/update.apk');
        break;
      case -1:
        // 下载失败，显示错误信息
        promptAction.showToast({ message: '下载失败: ' + msg });
        break;
    }
  }
);
```

### 2. 网络状态适配

结合网络状态管理优化下载策略：
```typescript
// 检查网络状态
const networkStatus = network.getCurrentType();
if (networkStatus === network.NetworkType.NETWORK_WIFI) {
  // WIFI环境直接下载
  startDownload();
} else if (networkStatus === network.NetworkType.NETWORK_MOBILE) {
  // 移动网络下询问用户
  promptAction.showDialog({
    message: '当前为移动网络，是否继续下载？',
    buttons: [{
      text: '取消'
    }, {
      text: '继续'
    }]
  }).then(result => {
    if (result.index === 1) {
      startDownload();
    }
  });
} else {
  promptAction.showToast({ message: '无网络连接' });
}
```

### 3. 断点续传实现思路

虽然当前框架未实现断点续传，但可基于现有代码扩展：
```typescript
// 扩展download方法支持range请求
async function downloadWithResume(url: string, filePath: string) {
  const fileSize = await FileUtil.getFileSize(filePath);
  const headers = fileSize > 0 ? {
    'Range': `bytes=${fileSize}-`
  } : {};

  // 使用headers发起部分请求
  // 下载时追加到文件而非覆盖
}
```

### 4. 下载任务管理

对于多文件下载场景，建议实现任务队列：
```typescript
class DownloadManager {
  private queue: DownloadTask[] = [];
  private currentTask: DownloadTask | null = null;

  addTask(task: DownloadTask) {
    this.queue.push(task);
    this.processQueue();
  }

  private processQueue() {
    if (this.currentTask || this.queue.length === 0) return;
    this.currentTask = this.queue.shift();
    this.startTask(this.currentTask);
  }

  // 其他实现...
}
```

## 总结与展望

ZeroOneApp的下载功能通过简洁而强大的设计，实现了文件下载的核心需求，具有以下特点：

1. **安全性**：下载前清理目标文件，避免版本冲突
2. **可靠性**：完整的错误处理和资源回收机制
3. **易用性**：简洁的API设计和清晰的状态码定义
4. **可扩展性**：模块化结构便于功能扩展

未来可考虑的优化方向：
- 实现断点续传，支持大文件分块下载
- 添加下载任务管理，支持暂停、继续、取消操作
- 增加网络类型判断和下载策略选择
- 实现下载速度限制，避免影响其他网络请求
- 添加文件校验机制，确保下载完整性

## 参考链接

- [HarmonyOS应用文件管理开发指导](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/app-file-access)
- [Axios拦截器与进度事件](https://ohpm.openharmony.cn/#/cn/detail/@ohos%2Faxios)