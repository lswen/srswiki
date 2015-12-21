#Bravo Media Server

BMS(Bravo Media Server)是基于SRS开发的商业版，由[chnvideo](http://www.chnvideo.com)研发。

## Major Compare

BMS和SRS的主要区别如下：

| Feature | SRS | BMS | Remark |
| ------  | --- | --- | ------ |
| 级别 | 工业级 | 工业级 | 发行版本都是工业级集群标准 |
| 开源 | 是 | 否 | BMS为闭源商业软件，提供售前咨询和售后服务，<br/>以及定制化开发，系统对接等 |
| 发行版 | 1.0 | 3.0 | SRS目前发行版为1.0，SRS2为alpha测试版。<br/>BMS合并了SRS2和SRS3的功能，为发行版 |
| 周期 | 1-2年 | 6个月 | SRS的版本发行周期为1到2年 |
| 代码量 | 9.95万行 | 11.29万行 | 包含服务器的注释和单元测试<br/>BMS比SRS多13.47%的代码量。 |
| 自动测试 | 无 | 支持 | BMS包含自动化测试系统，每次提交自动回归测试 |

## Detail Compare

BMS和SRS的详细对比如下（注意对比的是发行版，即SRS1和BMS3）。

以下是BMS和SRS都有的功能：

| Feature | SRS | BMS | Remark |
| ------  | ---- | ---- | ----- |
| HTTP回调 | 支持 | 支持 | 和外部业务系统对接 |
| 测速 | 支持 | 支持 | 支持服务器上行和下行速度测试 |
| TS矫正 | 支持 | 支持 | 支持时间戳矫正，避免重推和跳变引起播放器卡死 |
| Gop Cache | 支持 | 支持 | 缓存最近的Gop，让播放器快速启动(<0.1s) |
| Gop合并 | 支持 | 支持 | HLS按照GOP输出切片 |
| ATC | 支持 | 支持 | 支持绝对时间戳 |
| 边缘 | 支持 | 支持 | 源站和边缘组成流媒体分发集群 |
| 日志 | 支持 | 支持 | 提供可追溯的排错日志 |

以下为BMS增强的功能：

| Feature | SRS | BMS | Remark |
| ------  | ---- | ---- | ----- |
| 输入 | RTMP | RTMP, **_FLV, RTSP, UDP_**| 推流到服务器的输入方式 |
| 输出 | RTMP, HLS | RTMP, HLS, **_FLV, TS, MP3_** | 服务器分发给客户端的方式 |
| DVR | FLV文件 | **_对接观止收录系统_** | 支持录制RTMP到FLV文件 |
| 低延迟 | RTMP(3s+) | **RTMP/_FLV(1s+)_** | 低延迟模式 |
| 转码 | FFMPEG | **_对接观止转码云_** | 转码消耗非常多的系统资源 |
| HTTP API | 简版 | **_完善的API支持_** | API为服务器提供给外部的接口 |
| 下行并发 | 2.7K | **_7K_** | 下行RTMP/FLV的并发 |
| 上行并发 | 1.2K | **_4K_** | 上行推流RTMP的并发 |
| 热备 | RTMP | RTMP, **_HLS_** | 边缘在上层服务器故障时，切换到备用服务器 |

以下为BMS新增的功能：

| Feature | SRS | BMS | Remark |
| ------  | ---- | ---- | ----- |
| HLS内存模式 | 无 | **_支持_** | 支持内存模式分发HLS切片，避免磁盘IO |
| 时移 | 无 | **_对接观止时移系统_** | 在时移的基础上可以做高级收录和P2P |
| CDN预推 | 无 | **_支持_** | 将HLS预推到CDN节点 |
| 计费 | 无 | **_支持对接计费系统_** | 计费系统做定制和对接 |
| HLS纯音频 | 不支持 | **_支持MP3和AAC_** | HLS纯音频即没有视频流 |
| Vhost转换 | 不支持 | **_支持_** | 边缘回源，以及复杂业务系统需要转换Vhost |
| Kafka | 不支持 | **_支持_**| 对接到Kafka大数据集群 |
| 动态配置 | 不支持 | **_支持_** | 通过HTTP API从业务系统动态加载配置 |

## Contact

有需要请联系[chnvideo](http://www.chnvideo.com).