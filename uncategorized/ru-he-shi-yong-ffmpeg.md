---
description: How to use ffmpeg
---

# 如何使用ffmpeg

什么是[ffmpeg](https://ffmpeg.org)?\
ffmpeg is a universal media converter.\
ffmpeg是一个通用媒体格式转换器。它可以读取各种输入（包括实时抓取/录制设备）、过滤并将它们转码为多种输出格式。ffmpeg最令人赞叹的功能便是可以用它轻而易举的将不同媒体格式之间进行自由转换。只需指明输入和输出文件名， ffmpeg会从后缀名猜测媒体格式，同时适用于视频和音频文件。&#x20;

为什么俺使用ffmpeg？\
<mark style="color:red;">**开源、全能、小即是美**</mark>\
**命令格式**（注释：{}中是必选选项，\[]中是可选选项,实际用的时候不需要带这两种括号）

```
ffmpeg [global_options] {[input_file_options] -i input_url} ... {[output_file_options] output_url} ...
```

**warning**:所有没有使用 `-i` 指定的文件都被认为是输出文件;不应该将输入和输出混淆，先指定输入，再指定输出文件\
下面是一些常用命令的例子：\
获取媒体文件的信息

```
ffmpeg -i input_file_name
```

视频文件与音频文件均可

```
ffmpeg -i input_video_file.mp4 
ffmpeg -i input_audio_file.mp3
```

命令会输出很多与您文件无关的信息（ffmpeg本身的信息），你可以使用 `-hide_banner` 来隐藏掉它们。\
将媒体文件格式重编码为不同的格式，如将视频格式由mkv转换为mp4：

```
ffmpeg -i input_file.mkv output_file.mp4
```

在格式转化的同时，你可以设置输出文件的码率。如设置输出视频的码率为64kbit/s：

```
ffmpeg -i input_file.mkv -b:v 64k -bufsize 64k output_file.mp4
命令里的b是指bitrate（比特率）& v是指video(视频)
```

当然你也可以设置输出的视频文件的帧率。如设置帧率为30fps：

```
ffmpeg -i input_file.mkv -r 30 output_file.mp4
命令里的r应该是指rate
```

你甚至可以同时指定多个输出文件格式，当然ffmpeg也会同时输出多个文件：

```
ffmpeg -i input_file.avi output_1.mp4 output_2.mkv output_3.flv
```

ffmpeg还支持从视频文件中提取音频流或视频流\
如果你只想提取音频。如从mp4格式的视频文件中提取音频并设置音频的格式为mp3：

```
ffmpeg -i input_video_file.mp4 -vn input_audio_file.mp3
```

事实上只是在输出文件前加了参数`-vn`，v是指video(视频)，n是指no\
同理，如果你只想提取视频(无声仅有画面的视频)。如从mp4格式的视频文件中提取视频并设置视频的格式为mp4：

```
ffmpeg -i input_video_file.mp4 -an no-audio_video_file.mp4
```

此时，只是在输出文件前加了参数`-an`，a是指audio(音频)，n是指no\
需要说明的是，使用`-vn`会复用原有文件的比特率，一般来说，建议使用 `-ab` 或`-b:a`(音频比特率)来指定输出的音频文件的编码比特率：

```
ffmpeg -i input_video_file.mp4 -vn -ab 320k output_audio_file.mp3
```

以下是一些常用的比特率：

```
32kbit/s： 一般只适用于语音
96kbit/s： 一般用于语音或低质量流媒体
128或160kbit/s： 中等比特率质量
192kbit/s： 中等质量比特率
256kbit/s： 常用的高质量比特率
320kbit/s： MP3标准支持的最高水平
```

你还可以指定一些其他常用的参数，如 `-ar` (audio rate采样率: 22050, 441000, 48000), `-ac` (audio channels声道数), `-f` (format音频格式, 通常会自动识别)：

```
ffmpeg -i video.mp4 -vn -ar 48000 -ac 2 -b:a 256k -f mp3 audio.mp3
```

未完待续...
