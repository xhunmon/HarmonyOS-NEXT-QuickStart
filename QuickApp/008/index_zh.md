# 第8篇：常用工具篇（下）

在鸿蒙应用开发中，工具类的合理封装和复用是提升开发效率和代码健壮性的关键。本篇继续梳理项目中常用的实用工具，包括权限管理、日志打印、窗口与屏幕管理、系统API封装、日期格式化、音频处理等，所有API和术语均严格参考[华为开发者官网](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/development-intro-api)。

## 一、权限申请 —— permission

权限管理是保障应用安全和合规的基础。推荐使用`@kit.AbilityKit`的`abilityAccessCtrl`能力进行权限检测和动态申请。
- 检查权限：`await permission.checkPermission(Permissions.PERMISSION_NAME)`
- 申请权限：`await permission.requestPermission(Permissions.PERMISSION_NAME)`
- 回调方式：`permission.request(Permissions.PERMISSION_NAME, (agree) => { ... })`
- 权限需在`module.json5`声明。
- 相关API：[权限管理](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/app-permission-mgmt)

## 二、日志打印 —— log

日志是开发和运维的重要工具。推荐基于`@ohos.hilog`封装日志类，支持多级别（debug/info/warn/error/fatal）、动态关闭、设置打印等级等。
- 典型用法：`log.d(TAG, '调试信息')`
- 关闭日志：`log.closeLog()`
- 设置等级：`log.setLevel(hilog.LogLevel.INFO)`
- 相关API：[日志打印](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/hilog-guidelines-arkts)

## 三、窗口与屏幕管理 —— WindowUtil

用于监听和管理状态栏、导航栏、屏幕宽高变化，支持全屏、暗黑模式、系统栏显示隐藏等。
- 监听变化：`WindowUtil.listenToAppStorage(windowClass)`
- 设置暗黑模式：`WindowUtil.setDartEnable(windowClass, true)`
- 设置全屏：`WindowUtil.setFullScreenHideBar(windowClass)`
- 相关API：[窗口管理](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/application-window-stage)

## 四、系统API封装 —— sysUtil

以剪切板为例，封装常用系统API，提升开发效率。
- 复制文本：`sysUtil.copy('内容')`
- 相关API：[剪切板](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-pasteboard)

## 五、日期时间格式化 —— DateUtil

用于格式化时间戳、Date对象为常用字符串格式，支持自定义模板。
- 格式化：`DateUtil.formatDate(Date.now(), 'YYYY-MM-DD HH:mm:ss')`
- 相关API：[日期时间处理](https://developer.huawei.com/consumer/cn/doc/harmonyos-references/js-apis-intl)

## 六、PCM转WAV音频处理 —— wav

用于将PCM音频数据封装为WAV格式并保存到本地，适合录音等场景。
- 构造写入对象：`let wavWriter = new wav.Writer(...)`
- 写入PCM数据：`wavWriter.writeData(buffer)`
- 结束写入：`wavWriter.writeFinish()`
- 相关API：[文件操作](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/core-file-kit-intro)

```ts
/*
 * @Desc: 
 * @Author: qincji
 * @Date: 2024/5/4
 */
import fs from '@ohos.file.fs';
import { buffer } from '@kit.ArkTS';
import { Log } from 'common_utils';

export namespace wav {
  export class Header {
    mChunkID: string = "RIFF";
    mChunkSize: number = 0;
    mFormat: string = "WAVE";
    mSubChunk1ID: string = "fmt ";
    mSubChunk1Size: number = 16;
    mAudioFormat: number = 1; //short
    mNumChannel: number = 1; //short
    mSampleRate: number = 8000;
    mByteRate: number = 0; //
    mBlockAlign: number = 0; //short
    mBitsPerSample: number = 8; //short

    mSubChunk2ID: string = "data";
    mSubChunk2Size: number = 0;

    constructor(sampleRateInHz: number, channels: number, bitsPerSample: number) {
      this.mSampleRate = sampleRateInHz;
      this.mBitsPerSample = bitsPerSample;
      this.mNumChannel = channels;
      this.mByteRate = this.mSampleRate * this.mNumChannel * this.mBitsPerSample / 8;
      this.mBlockAlign = (this.mNumChannel * this.mBitsPerSample / 8);
    }
  }

  function deleteIfExit(path: string) {
    if (fs.accessSync(path)) {
      fs.unlinkSync(path)
    }
  }


  class Options {
    offset?: number;
    length?: number;
  }

  const MAX_COPY_LEN: number = 1024 * 10;
  const TAG = 'WavUtil';

  function offset(offset: number, length: number): Options {
    return {
      offset: offset,
      length: length
    }
  }

  export class Writer {
    private header: Header;
    private tempPath: string;
    private filePath: string;
    private temp: fs.File;
    private file: fs.File;
    private dataLength: number = 0;

    constructor(folder: string, filepath: string, sampleRateInHz: number, channels: number, bitsPerSample: number) {
      this.header = new Header(sampleRateInHz, channels, bitsPerSample);
      this.tempPath = folder + '/temp.wav';
      this.filePath = filepath;
      deleteIfExit(filepath);
      deleteIfExit(this.tempPath);
      this.file = fs.openSync(filepath, fs.OpenMode.READ_WRITE | fs.OpenMode.CREATE);
      this.temp = fs.openSync(this.tempPath, fs.OpenMode.READ_WRITE | fs.OpenMode.CREATE);
    }

    
    public getDuration(): number {
      return Number((this.dataLength * 1000 / this.header.mByteRate).toFixed(0));
    }

    
    public creatNewFile(filepath: string) {
      deleteIfExit(this.filePath);
      this.filePath = filepath;
      this.file = fs.openSync(filepath, fs.OpenMode.READ_WRITE | fs.OpenMode.CREATE);
    }

    public clearCache() {
      fs.closeSync(this.temp.fd);
      fs.closeSync(this.file.fd);
      deleteIfExit(this.filePath);
      deleteIfExit(this.tempPath);
    }

    public writeData(buffer: ArrayBuffer) {
      fs.writeSync(this.temp.fd, buffer, {
        offset: this.dataLength,
        length: buffer.byteLength
      });
      this.dataLength += buffer.byteLength;
      // Log.d(TAG, () => 'dataLength： ' + this.dataLength);
    }

    
    public writeFinish(clearEndSecond: number = 0) {
      Log.d(TAG, () => 'writeFinish dataLength： ' + this.dataLength);
      // this.dataLength = this.dataLength - clearEndSecond * this.header.mByteRate;
      this.writeHeader();
      const HEADER_SIZE: number = 44;
      let bufferSize: number = HEADER_SIZE; //这是写入头部后的大小
      let buffer: ArrayBuffer = new ArrayBuffer(MAX_COPY_LEN);
      let curSize = fs.readSync(this.temp.fd, buffer);
      while (curSize > 0) {
        fs.writeSync(this.file.fd, buffer, {
          offset: bufferSize,
          length: curSize
        });
        bufferSize += curSize;
        if (curSize < MAX_COPY_LEN || bufferSize - HEADER_SIZE >= this.dataLength) { 
          break;
        }
        curSize = fs.readSync(this.temp.fd, buffer);
      }

      fs.closeSync(this.temp.fd);
      fs.closeSync(this.file.fd);
      fs.unlinkSync(this.tempPath);

      Log.d(TAG, () => 'finish...');
    }

    private writeHeader() {
      fs.writeSync(this.file.fd, buffer.from(this.header.mChunkID).buffer, offset(0, 4));
      const fileSize = fs.statSync(this.temp.fd).size;
      const fileSizeArray = [(fileSize - 8) & 0xff, ((fileSize - 8) >> 8) & 0xff,
        ((fileSize - 8) >> 16) & 0xff, ((fileSize - 8) >> 24) & 0xff];
      fs.writeSync(this.file.fd, buffer.from(fileSizeArray).buffer, offset(4, 4));
      fs.writeSync(this.file.fd, buffer.from(this.header.mFormat).buffer, offset(8, 4));
      fs.writeSync(this.file.fd, buffer.from(this.header.mSubChunk1ID).buffer, offset(12, 4));
      const subChunkArray: number[] = [16, 0, 0, 0];
      fs.writeSync(this.file.fd, buffer.from(subChunkArray).buffer, offset(16, 4));
      const audioFormatArray: number[] = [1, 0];
      fs.writeSync(this.file.fd, buffer.from(audioFormatArray).buffer, offset(20, 2));
      const channelArray: number[] = [this.header.mNumChannel, 0];
      fs.writeSync(this.file.fd, buffer.from(channelArray).buffer, offset(22, 2));
      const sampleR = this.header.mSampleRate;
      const sampleRateArray = [sampleR & 0xff, (sampleR >> 8) & 0xff, (sampleR >> 16) & 0xff, (sampleR >> 24) & 0xff];
      fs.writeSync(this.file.fd, buffer.from(sampleRateArray).buffer, offset(24, 4));
      const bitRate = this.header.mByteRate;
      const bitRateArray = [bitRate & 0xff, (bitRate >> 8) & 0xff, (bitRate >> 16) & 0xff, (bitRate >> 24) & 0xff];
      fs.writeSync(this.file.fd, buffer.from(bitRateArray).buffer, offset(28, 4));
      const blockArray = [this.header.mBlockAlign, 0];
      fs.writeSync(this.file.fd, buffer.from(blockArray).buffer, offset(32, 2));
      const bitPerRateArray = [this.header.mBitsPerSample, 0];
      fs.writeSync(this.file.fd, buffer.from(bitPerRateArray).buffer, offset(34, 2));
      fs.writeSync(this.file.fd, buffer.from(this.header.mSubChunk2ID).buffer, offset(36, 4));
      const dataSize = this.dataLength;
      const dataSizeArray = [dataSize & 0xff, (dataSize >> 8) & 0xff, (dataSize >> 16) & 0xff, (dataSize >> 24) & 0xff];
      fs.writeSync(this.file.fd, buffer.from(dataSizeArray).buffer, offset(40, 4));
    }
  }
}

```

## 七、最佳实践与官方文档

- 工具类建议单一职责、接口简洁，便于复用和维护。
- 重要能力如权限、日志、窗口管理等务必参考[官方文档](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/arkts-coding-style-guide)和[ArkTS编程规范](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides-V5/arkts-coding-style-guide-V5)。
- 代码应有详细注释，关键异常需日志记录，便于排查。

**到这里，所有的内容已经结束了！本章的完整源码已经上传到gitee了：[鸿蒙应用0-1开发](https://gitee.com/qincji/ZeroOneApp)。**