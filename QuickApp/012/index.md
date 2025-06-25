# ZeroOneApp Network Download Function Implementation and Optimization

## Introduction

In mobile application development, file downloading is a common and critical feature involving network requests, progress monitoring, file operations, and exception handling. ZeroOneApp's network module not only provides basic HTTP request capabilities but also encapsulates an efficient and reliable download functionality that supports progress feedback, file management, and error recovery. This article will deeply analyze the download implementation details in the `NetRequest` class, explore its design ideas and optimization strategies, providing reference for implementing download functionality in HarmonyOS applications.

## Official Resources

- **HarmonyOS File Management**: Provides complete APIs for file operations within the application sandbox, including creating, deleting, reading, and writing files [HarmonyOS File Management Documentation](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/app-file-access)
- **Axios Download Progress**: The `@ohos/axios` library supports obtaining download progress information through the `onDownloadProgress` callback [Axios Request Configuration](https://ohpm.openharmony.cn/#/cn/detail/@ohos%2Faxios)
- **HarmonyOS Network Status Management**: Provides network connection status monitoring capabilities that can be used to optimize download experience [HarmonyOS Network Status](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/http-request)

## Detailed Explanation

### 1. Core Implementation of Download Function

ZeroOneApp's download functionality is implemented through the `download` method, which encapsulates the complete lifecycle of file downloading, including request configuration, progress monitoring, file processing, and result callbacks:

```typescript:common/net/src/main/ets/core/NetRequest.ets
download(url: string, filePath: string, onResult: (status: number, progress: number, msg: string) => void) {
  log.i(TAG, `[Download][${url}]: ${filePath}`);
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
      onResult(2, progress, 'Downloading');
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
    log.e(TAG, `[Exception][${url}]: ${StringUtil.toStr(error)}`);
    FileUtil.delete(filePath);
    onResult(-1, 0, error.message);
  })
}
```

### 2. Key Technical Points Analysis

#### 2.1 File Path Handling and Conflict Resolution

The target path file is deleted before downloading to avoid interference from old files and path conflicts:
```typescript
FileUtil.delete(filePath);
```
This operation ensures that each download starts fresh, preventing verification errors or version confusion caused by file residues.

#### 2.2 Progress Monitoring Implementation

Real-time download progress is obtained through Axios's `onDownloadProgress` callback:
```typescript
onDownloadProgress: (progressEvent: AxiosProgressEvent): void => {
  if (progressEvent === undefined || progressEvent.total === undefined) {
    return;
  }
  const progress = progressEvent.total ? Math.ceil(progressEvent.loaded / progressEvent.total * 100) : 0;
  onResult(2, progress, 'Downloading');
}
```
Progress calculation uses the formula `loaded/total*100` with `Math.ceil` to ensure progress values are integer percentages for easy UI display.

#### 2.3 Status Code Design

Download status is传递 through the `status` parameter of the callback function, defining four clear states:
- `-1`: Download failed
- `0`: Not downloaded (initial state)
- `1`: Download successful
- `2`: Downloading
This state design enables the caller to easily handle UI display and business logic for different download phases.

#### 2.4 Error Handling and Resource Cleanup

In case of download failure, the framework automatically deletes incomplete files to avoid occupying storage space:
```typescript
.catch((error: AxiosError) => {
  log.e(TAG, `[Exception][${url}]: ${StringUtil.toStr(error)}`);
  FileUtil.delete(filePath);
  onResult(-1, 0, error.message);
})
```
This mechanism ensures the cleanliness of the file system and prevents accumulation of invalid files.

## Practical Recommendations

### 1. Complete Download Function Usage Example

```typescript
// Download APK update package
net.download(
  'https://example.com/xxx.zip',
  getContext().filesDir + '/update.apk',
  (status, progress, msg) => {
    switch (status) {
      case 2:
        // Update progress bar
        this.downloadProgress = progress;
        break;
      case 1:
        // Download successful, install APK
        this.installApk(getContext().filesDir + '/update.apk');
        break;
      case -1:
        // Download failed, display error message
        promptAction.showToast({ message: 'Download failed: ' + msg });
        break;
    }
  }
);
```

### 2. Network Status Adaptation

Optimize download strategy by combining with network status management:
```typescript
// Check network status
const networkStatus = network.getCurrentType();
if (networkStatus === network.NetworkType.NETWORK_WIFI) {
  // Direct download in WIFI environment
  startDownload();
} else if (networkStatus === network.NetworkType.NETWORK_MOBILE) {
  // Ask user in mobile network
  promptAction.showDialog({
    message: 'Currently using mobile data, continue downloading?',
    buttons: [{
      text: 'Cancel'
    }, {
      text: 'Continue'
    }]
  }).then(result => {
    if (result.index === 1) {
      startDownload();
    }
  });
} else {
  promptAction.showToast({ message: 'No network connection' });
}
```

### 3. Resumable Download Implementation Idea

Although the current framework does not implement resumable download, it can be extended based on existing code:
```typescript
// Extend download method to support range requests
async function downloadWithResume(url: string, filePath: string) {
  const fileSize = await FileUtil.getFileSize(filePath);
  const headers = fileSize > 0 ? {
    'Range': `bytes=${fileSize}-`
  } : {};

  // Use headers to initiate partial request
  // Append to file instead of overwriting when downloading
}
```

### 4. Download Task Management

For multi-file download scenarios, it is recommended to implement a task queue:
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

  // Other implementations...
}
```

## Summary and Outlook

ZeroOneApp's download functionality implements core file download requirements through a concise yet powerful design, featuring:

1. **Security**: Cleans target files before downloading to avoid version conflicts
2. **Reliability**: Complete error handling and resource recovery mechanisms
3. **Usability**: Concise API design and clear status code definition
4. **Extensibility**: Modular structure facilitating function expansion

Future optimization directions to consider:
- Implement resumable download to support chunked downloading of large files
- Add download task management supporting pause, resume, and cancel operations
- Increase network type judgment and download strategy selection
- Implement download speed limitation to avoid affecting other network requests
- Add file verification mechanism to ensure download integrity

## References

- [HarmonyOS Application File Management Development Guide](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/app-file-access)
- [Axios Interceptors and Progress Events](https://ohpm.openharmony.cn/#/cn/detail/@ohos%2Faxios)