title: php获取视频文件编码信息
date: 2014-12-05 21:57:58
tags:
- ffmpeg
- php
---

## 使用ffmpeg
```
FFmpeg是一个自由软件，可以运行音频和视频多种格式的录影、转换、流功能[1]，
包含了libavcodec ─这是一个用于多个项目中音频和视频的解码器库，以及libavformat——一个音频与视频格式转换库。
```

* 安装ffmpeg

```shell
#rpm -ivh http://apt.sw.be/redhat/el6/en/i386/rpmforge/RPMS/rpmforge-release-0.5.3-1.el6.rf.i686.rpm

#yum install ffmpeg ffmpeg-devel

#ffmpeg -v
FFmpeg version 0.6.5, Copyright (c) 2000-2010 the FFmpeg developers
```

* 使用命令行获取文件信息
```shell
# ffmpeg -i 庞麦郎_-_我的滑板鞋.mp4
Input #0, mov,mp4,m4a,3gp,3g2,mj2, from '庞麦郎_-_我的滑板鞋.mp4':
  Metadata:
    major_brand     : isom
    minor_version   : 512
    compatible_brands: isomiso2avc1mp41
    encoder         : Lavf54.6.100
  Duration: 00:03:36.48, start: 0.000000, bitrate: 2778 kb/s
    Stream #0.0(und): Video: h264, yuv420p, 1920x1080 [PAR 1:1 DAR 16:9], 2674 kb/s, 25 fps, 25 tbr, 25025 tbn, 50 tbc
    Stream #0.1(und): Audio: aac, 44100 Hz, stereo, s16, 97 kb/s
At least one output file must be specified
```
* 使用正则提取需要的信息
```php
<?php
function getVideoFormat($file)
{
    $ffmpeg = "/usr/bin/ffmpeg";

    ob_start();
    passthru(sprintf($ffmpeg.' -i "%s" 2>&1', $file));
    $info = ob_get_contents();
    ob_end_clean();
    preg_match('/Video:\s(?<format>.*?),/', $info, $match);
    return $match;
}

$file = __DIR__ . '/庞麦郎_-_我的滑板鞋.mp4';
var_dump(getVideoFormat($file));

/*
array(3) {
  [0]=>
  string(12) "Video: h264,"
  ["format"]=>
  string(4) "h264"
  [1]=>
  string(4) "h264"
}
*/
```

## 使用[getID3][1]类库

```php
<?php
require_once('../getid3/getid3.php');

// Initialize getID3 engine
$getID3 = new getID3;

$filename =  __DIR__ .'/../庞麦郎_-_我的滑板鞋.mp4';
// Analyze file and store returned data in $ThisFileInfo
$ThisFileInfo = $getID3->analyze($filename);
print_r($ThisFileInfo['video']);
/*
Array
(
    [dataformat] => quicktime
    [resolution_x] => 1920
    [resolution_y] => 1080
    [fourcc] => avc1
    [frame_rate] => 25
)
*/
```
  [1]: http://getid3.sourceforge.net/
