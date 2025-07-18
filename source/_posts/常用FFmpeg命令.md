---
title: 常用ffmpeg命令
tags: [ffmpeg]
categories:
  - [ffmpeg]
date: 2025-07-18 10:05:30
---

# 常用FFmpeg命令

## 一、格式转换

### 常用示例

1. **将 MP4 转换为 AVI 格式：**

   ```
   $ ffmpeg -i input.mp4 output.avi
   ```


2. **将 WebM 转换为 MP4 格式：**

   ```
   $ ffmpeg -i movie.webm movie.mp4
   ```

3.  **m3u8与MP4格式互转：**

   ```
   $ ffmepg -i http://localhost:8000/input.m3u8 -c copy output.mp4
   
   # hls_time 切片时间间隔
   # 切片时间最优
   $ ffmpeg -i input.mp4 -c:v libx264 -c:a aac -hls_time \
   10 -hls_list_size 0 -hls_segment_filename "output_%03d.ts" output.m3u8
   
   # -crf 18: 恒定质量模式,值越小质量越高(18~23是高质量范围, 18接近无损)
   # -preset slow: 编码速度与压缩率的平衡,slow比medium压缩率更高,质量更好（但更慢
   # -b:a 192k: 音频码率设为 192 kbps(比默认128k更高, 提升音质)
   $ ffmpeg -i input.mp4 -c:v libx264 -crf 18 -preset slow -c:a aac -b:a 192k -hls_time 5 -hls_list_size 0 -hls_segment_filename "output_%03d.ts" output.m3u8
   ```
## 二、FFmpeg 全局与主要选项

- • **-y**：覆盖输出文件
- • **-n**：不要覆盖输出文件
- • **-f fmt**：强制输入或输出文件格式
- • **-c codecName**：指定编解码器, 适用于输入和输出文件

## 三、基础命令

1. **列出可用编码格式：**

   ```
   $ ffmpeg -codecs
   ```

2. **指定输入文件和编码器：**

   ```
   $ ffmpeg -i input.mp4 -c:v libx264 -c:a aac output.mp4
   ```

## 四、视频参数调节

1. **设置比特率**：指定视频比特率，单位 kbit/s

   ```
   $ ffmpeg -i input.avi -b:v 64k -bufsize 64k output.avi
   ```

2. **指定分辨率与帧率：**

   ```
   $ ffmpeg -i input.avi -s 1280x720 -r 30 output.avi
   ```

## 五、视频编辑与剪辑

1. 剪切视频片段

```
# 将视频从指定时间开始，剪切一段视频：
$ ffmpeg -i input.mp4 -ss 00:01:45 -t 00:02:35 -c copy output.mp4
```

2. 调整视频帧速率

```
# 强制视频帧率为 24 fps：
$ ffmpeg -i input.avi -r 24 output.avi
```

## 六、音频参数调节

1. **设置音频比特率：**

   ```
   $ ffmpeg -i input.mp4 -ab 128k output.mp3
   ```

2. **修改音量：**

   ```
   # 增加音量
   $ ffmpeg -i input.mov -filter:a "volume=1.5" output.mov  
   ```

## 七、特定任务场景

### 1. 转换为 GIF 动图

截取视频片段生成 GIF：

```
$ ffmpeg -ss 2 -t 5 -i input.mp4 -vf "fps=10,scale=320:-1" output.gif
```

### 2. 提取视频中的音频

 从视频中提取音频并保存为 mp3 格式：

   ```
   $ ffmpeg -i input.mp4 -vn -c copy output.mp3
   ```

### 3. 合并多个视频文件

将多个同规格视频合并到一个文件中（确保视频格式一致）：

```
# 列表文件 videos.txt 格式示例
file '1.mp4'
file '2.mp4'
file '3.mp4'

# 合并命令
$ ffmpeg -f concat -i videos.txt -c copy output.mp4
```

## 八、视频压缩与编码

#### H265 双重编码

```
$ ffmpeg -y -i input.mp4 -c:v libx265 -b:v 2600k -pass 1 -f mp4 /dev/null && \
  ffmpeg -i input.mp4 -c:v libx265 -b:v 2600k -pass 2 output.mp4
```

## 九、字幕处理

### 1. 将字幕文件嵌入视频

```
$ ffmpeg -i input.mov -filter:v "subtitles=subtitles.srt" output.mov
```

### 2. 字幕格式转换

```
$ ffmpeg -i subtitle.srt subtitle.ass  # SRT 转 ASS
```

## 十、录屏操作

### MacOS 录屏

```
$ ffmpeg -f avfoundation -i 1:0 -preset ultrafast screen_recording.mkv
```

### Windows 录屏

```
$ ffmpeg -hide_banner -loglevel error -stats -f gdigrab -framerate 60 \
-offset_x 0 -offset_y 0 -video_size 1920x1080 -draw_mouse 1 -i deskop \
-c:v libx264 -r 60 -preset ultrafast -pix_fmt yuv420p -y screen_record.mp4
```



# ffprobe常见命令

```
# 获取input.mp4的编码格式 eg. h264、hevc.
$ ffprobe -v error -select_streams v:0 -show_entries stream=codec_name -of default=noprint_wrappers=1:nokey=1 input.mp4
```

```
# 获取input.mp4音频编码格式 eg. aac、mp3
$ ffprobe -v error -select_streams a:0 -show_entries stream=codec_name -of default=noprint_wrappers=1:nokey=1 input.mp4
```

```
# 获取关键帧间隔
$ ffprobe -v error -select_streams v:0 -show_entries frame=pict_type -of csv input.mp4 | grep -c "I"
```

