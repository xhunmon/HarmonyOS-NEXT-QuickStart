# Part 8: Common Utility Tools (Part 2)

In HarmonyOS app development, proper encapsulation and reuse of utility classes are key to improving development efficiency and code robustness. This article continues to summarize practical utility tools commonly used in projects, including permission management, logging, window and screen management, system API encapsulation, date formatting, and audio processing. All APIs and terminology strictly follow the [Huawei Developer Official Documentation](https://developer.huawei.com/consumer/en/doc/harmonyos-references/development-intro-api).

## 1. Permission Management — permission

Permission management is fundamental to ensuring application security and compliance. It is recommended to use the `abilityAccessCtrl` capability from `@kit.AbilityKit` for permission checking and dynamic requests.
- Check permission: `await permission.checkPermission(Permissions.PERMISSION_NAME)`
- Request permission: `await permission.requestPermission(Permissions.PERMISSION_NAME)`
- Callback style: `permission.request(Permissions.PERMISSION_NAME, (agree) => { ... })`
- Permissions must be declared in `module.json5`.
- Related API: [Permission Management](https://developer.huawei.com/consumer/en/doc/harmonyos-guides/app-permission-mgmt)

## 2. Logging — log

Logging is an important tool for development and operations. It is recommended to encapsulate the log class based on `@ohos.hilog`, supporting multiple levels (debug/info/warn/error/fatal), dynamic disabling, and log level settings.
- Typical usage: `log.d(TAG, 'Debug info')`
- Disable logging: `log.closeLog()`
- Set log level: `log.setLevel(hilog.LogLevel.INFO)`
- Related API: [Logging Guidelines](https://developer.huawei.com/consumer/en/doc/harmonyos-guides/hilog-guidelines-arkts)

## 3. Window and Screen Management — WindowUtil

Used to listen for and manage changes in the status bar, navigation bar, and screen dimensions. Supports full screen, dark mode, and system bar show/hide.
- Listen for changes: `WindowUtil.listenToAppStorage(windowClass)`
- Set dark mode: `WindowUtil.setDartEnable(windowClass, true)`
- Set full screen: `WindowUtil.setFullScreenHideBar(windowClass)`
- Related API: [Window Management](https://developer.huawei.com/consumer/en/doc/harmonyos-guides/application-window-stage)

## 4. System API Encapsulation — sysUtil

Taking the clipboard as an example, encapsulate common system APIs to improve development efficiency.
- Copy text: `sysUtil.copy('content')`
- Related API: [Clipboard](https://developer.huawei.com/consumer/en/doc/harmonyos-references/js-apis-pasteboard)

## 5. Date and Time Formatting — DateUtil

Used to format timestamps or Date objects into common string formats, supporting custom templates.
- Format: `DateUtil.formatDate(Date.now(), 'YYYY-MM-DD HH:mm:ss')`
- Related API: [Date and Time Handling](https://developer.huawei.com/consumer/en/doc/harmonyos-references/js-apis-intl)

## 6. PCM to WAV Audio Processing — wav

Used to encapsulate PCM audio data into WAV format and save it locally, suitable for recording scenarios.
- Construct writer object: `let wavWriter = new wav.Writer(...)`
- Write PCM data: `wavWriter.writeData(buffer)`
- Finish writing: `wavWriter.writeFinish()`
- Related API: [File Operations](https://developer.huawei.com/consumer/en/doc/harmonyos-guides/core-file-kit-intro)

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

## 7. Best Practices and Official Documentation

- Utility classes should follow the single responsibility principle and have simple interfaces for easy reuse and maintenance.
- For important capabilities such as permissions, logging, and window management, always refer to the [Official Documentation](https://developer.huawei.com/consumer/en/doc/harmonyos-guides/arkts-coding-style-guide) and [ArkTS Coding Style Guide](https://developer.huawei.com/consumer/en/doc/harmonyos-guides-V5/arkts-coding-style-guide-V5).
- Code should include detailed comments, and key exceptions should be logged for troubleshooting.

**All content for this chapter is complete! The full source code has been uploaded to Gitee: [HarmonyOS App 0-1 Development](https://gitee.com/qincji/ZeroOneApp).** 