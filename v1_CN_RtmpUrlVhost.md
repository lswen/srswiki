# RTMP的URL/Vhost规则

RTMP的url其实很简单，vhost其实也没有什么新的概念，但是对于没有使用过的同学来讲，还是很容易混淆。几乎每个新人都必问的问题：RTMP那个URL推流时应该填什么，什么是vhost，什么是app？

RTMP和HLS的优势参考：[HLS](https://github.com/winlinvip/simple-rtmp-server/wiki/v1_CN_DeliveryHLS)

## 应用场景

Vhost的主要应用场景包括：
* 一个分发网络支持多个客户：譬如CDN，一个分发网络中，有N个客户公用一套流媒体系统，如何区分用户，计费，监控等等？通过app么？大家可能都叫做live之类。最好是通过各自的域名。
* 不同的应用配置：譬如FMLE推上来的流是h264+mp3，可以将音频转码后放到其他的vhost分发hls，这样接入h264+mp3的vhost就不用切hls。

总之，vhost作为应用配置的单元，能隔离客户，应用不同的配置。

## 标准RTMP URL

标准RTMP URL指的是最大兼容的RTMP URL，基本上所有的服务器和播放器都能识别的URL，和HTTP URL其实很相似，例如：

<table>
<thead>
<tr>
<th>HTTP</th>
<th>Schema</th>
<th>Host</th>
<th>Port</th>
<th colspan=2>Path</th>
</tr>
</thead>
<tbody>
<tr>
<td>http://192.168.1.10:80/players/srs_player.html</td>
<td>http</td>
<td>192.168.1.10</td>
<td>80</td>
<td colspan=2>/players/srs_player.html</td>
</tr>
<tr>
<td>rtmp://192.168.1.10:1935/live/livestream</td>
<td>rtmp</td>
<td>192.168.1.10</td>
<td>1935</td>
<td>live</td>
<td>livestream</td>
</tr>
</tbody>
<tfoot>
<tr>
<th>RTMP</th>
<th>Schema</th>
<th>Host</th>
<th>Port</th>
<th>App</th>
<th>Stream</th>
</tr>
</tfoot>
</table>

其中：
* Schema：协议头，HTTP为HTTP或HTTPS，RTMP为RTMP/RTMPS/RTMPE/RTMPT等众多协议，还有新出的RTMFP。
* Host：主机，表示要连接的主机，可以为主机DNS名称或者IP地址。商用时，一般不会用IP地址，而是DNS名称，这样可以用CDN分发内容（CDN一般使用DNS调度，即不同网络和地理位置的用户，通过DNS解析到的IP不一样，实现用户的就近访问）。
* Port：端口，HTTP默认为80，RTMP默认为1935。当端口没有指定时，使用默认端口。
* Path：路径，HTTP访问的文件路径。
* App：RTMP的Application（应用）名称，可以类比为文件夹。以文件夹来分类不同的流，没有特殊约定，可以任意划分。
* Stream：RTMP的Stream（流）名称，可以类比为文件。

## NoVhost

其实，vhost大多数用户都用不到，而且不推荐用，有点复杂。一般的用户用app就可以了。因为vhost/app/stream，只是一个分类方法而已；vhost需要在配置文件中说明，app/stream都不需要配置。

什么时候用vhost？如果你是提供服务，譬如你有100个客户，都要用一套平台，走同样的流媒体服务器分发。那可以每个客户一个vhost，这样他们的app和stream可以相同都可以。

一般的用法，举个例子，有个视频网站，自己搭建服务器，所以只有他自己一个客户，就不要用vhost了。假设视频网站提供聊天服务，聊天有不同的话题类型，譬如：军事栏目，读书栏目，历史栏目三个分类，每个分类下面有很多聊天室。只要这么配置就好：

```bash
listen              1935;
vhost __defaultVhost__ {
}
```

生成网页时，譬如军事栏目的网页，都用app名称为`military`，某个聊天室叫做`火箭`，这个页面的流可以用：`rtmp://yourdomain.com/military/rock`，编码器也推这个流，所有观看这个`军事栏目/火箭`聊天室的页面的人，都播放这个流。

军事栏目另外的网页，都用同样的app名称`military`，但是流不一样，譬如某个聊天室叫做`雷达`，这个页面的流可以用：`rtmp://yourdomain.com/military/radar`，推流和观看一样。

如此类推，军事栏目页面生成时，不用更改srs的任何配置。也就是说，新增聊天室，不用改服务器配置；新增分类，譬如加个`公开课`的聊天室，也不用改服务器配置。足够简单！

另外，读书栏目可以用app名称为`reader`，栏目下的某个聊天室叫`红楼梦`，这个页面的流可以用：`rtmp://yourdomain.com/reader/red_mansion`，所有在这个聊天室的人都是播放这个流。

## Vhost的应用

RTMP的Vhost和HTTP的Vhost概念是一样的：虚拟主机。详见下表（假设域名demo.srs.com被解析到IP为192.168.1.10的服务器）：

<table>
<thead>
<tr>
<th>HTTP</th>
<th>Host</th>
<th>Port</th>
<th>VHost</th>
</tr>
</thead>
<tbody>
<tr>
<td>http://demo.srs.com:80/players/srs_player.html</td>
<td>192.168.1.10</td>
<td>80</td>
<td>demo.srs.com</td>
</tr>
<tr>
<td>rtmp://demo.srs.com:1935/live/livestream</td>
<td>192.168.1.10</td>
<td>1935</td>
<td>demo.srs.com</td>
</tr>
</tbody>
<tfoot>
<tr>
<th>RTMP</th>
<th>Host</th>
<th>Port</th>
<th>VHost</th>
</tr>
</tfoot>
</table>

Vhost主要的作用是：
* 支持多用户：当一台服务器需要服务多个客户，譬如CDN有cctv（央视）和wasu（华数传媒）两个客户时，如何隔离他们两个的资源？相当于不同的用户共用一台计算机，他们可以在自己的文件系统建立同样的文件目录结构，但是彼此不会冲突。
* 域名调度：CDN分发内容时，需要让用户访问离自己最近的边缘节点，边缘节点再从源站或上层节点获取数据，达到加速访问的效果。一般的做法就是Host是DNS域名，这样可以根据用户的信息解析到不同的节点。
* 支持多配置：有时候需要使用不同的配置，考虑一个支持多终端（PC/Apple/Android）的应用，PC上RTMP分发，Apple和Android是HLS分发，如何让PC延迟最低，同时HLS也能支持，而且终端播放时尽量地址一致（降低终端开发难度）？可以使用两个Vhost，PC和HLS；PC配置为最低延迟的RTMP，并且将流转发给HLS的Vhost，可以对音频转码（可能不是H264/AAC）后切片为HLS。PC和HLS这两个Vhost的配置肯定是不一样的，播放时，流名称是一样，只需要使用不同的Host就可以。

### Vhost支持多用户

假设cctv和wasu都运行在一台边缘节点(192.168.1.10)上，用户访问这两个媒体的流时，Vhost的作用见下表：

<table>
<thead>
<tr>
<th>RTMP</th>
<th>Host</th>
<th>Port</th>
<th>VHost</th>
<th>App</th>
<th>Stream</th>
</tr>
</thead>
<tbody>
<tr>
<td>rtmp://show.cctv.cn/live/livestream</td>
<td>192.168.1.10</td>
<td>1935</td>
<td>show.cctv.cn</td>
<td>live</td>
<td>livestream</td>
</tr>
<tr>
<td>rtmp://show.wasu.cn/live/livestream</td>
<td>192.168.1.10</td>
<td>1935</td>
<td>show.wasu.cn</td>
<td>live</td>
<td>livestream</td>
</tr>
</tbody>
</table>

在边缘节点（192.168.1.10）上的SRS，需要配置Vhost，例如：

```bash
listen              1935;
vhost show.cctv.cn {
}
vhost show.wasu.cn {
}
```

### Vhost域名调度

详细参考DNS和CDN的实现。

### Vhost支持多配置

以上面举的例子，若cctv需要延迟最低（意味着启动时只有声音，画面是黑屏），而wasu需要快速启动（打开就能看到视频，服务器cache了最后一个gop，延迟会较大）。

只需要对这两个Vhost进行不同的配置，例如：

```bash
listen              1935;
vhost show.cctv.cn {
    gop_cache       off;
}
vhost show.wasu.cn {
    gop_cache       on;
}
```

总之，这两个Vhost的配置完全没有关系，不会相互影响。

## \_\_defaultVhost\_\_

FMS的\_\_defaultVhost\_\_是默认的vhost，当用户请求的vhost没有匹配成功时，若配置了defaultVhost，则使用它来提供服务。若匹配失败，也没有defaultVhost，则返回错误。

譬如，服务器192.168.1.10上的SRS配置如下：

```bash
listen              1935;
vhost demo.srs.com {
    enabled         on;
}
```

那么，当用户访问以下vhost时：
* rtmp://demo.srs.com/live/livestream：成功，匹配vhost为demo.srs.com
* rtmp://192.168.1.10/live/livestream：失败，没有找到vhost，也没有defaultVhost。

defaultVhost和其他vhost的规则一样，只是用来匹配那些没有匹配成功的vhost的请求的。

## 访问指定的Vhost

如何访问某台服务器上的Vhost？有两个方法：
* 配置hosts：因为Vhost实际上就是DNS解析，所以可以配置客户端的hosts，将域名（Vhost）解析到指定的服务器，就可以访问这台服务器上的指定的vhost。
* 使用app的参数：需要服务器支持。在app后面带参数指定要访问的Vhost。SRS支持?vhost=VHOST和...vhost...VHOST这两种方式，后面的方式是避免一些播放器不识别？和=等特殊字符。

普通用户不用这么麻烦，直接访问RTMP地址就好了，有时候运维需要看某台机器上的Vhost的流是否有问题，就需要这种特殊的访问方式。考虑下面的例子：

```bash
RTMP URL: rtmp://demo.srs.com/live/livestream
边缘节点数目：50台
边缘节点IP：192.168.1.100 至 192.168.1.150
边缘节点SRS配置：
    listen              1935;
    vhost demo.srs.com {
        mode remote;
        origin: xxxxxxx;
    }
```

各种访问方式见下表：

<table>
<thead>
<tr>
<th>用户</th>
<th>RTMP URL</th>
<th>hosts设置</th>
<th>目标</th>
</tr>
</thead>
<tbody>
<tr>
<td>普通用户</td>
<td>rtmp://demo.srs.com/live/livestream</td>
<th>无</th>
<td>由DNS<br/>解析到指定边缘</td>
</tr>
<tr>
<td>运维</td>
<td>rtmp://demo.srs.com/live/livestream</td>
<th>192.168.1.100 demo.srs.com</th>
<td>查看192.168.1.100上的流</td>
</tr>
<tr>
<td>运维</td>
<td>rtmp://192.168.1.100/live?<br/>vhost=demo.srs.com/livestream</td>
<th>无</th>
<td>查看192.168.1.100上的流</td>
</tr>
<tr>
<td>运维</td>
<td>rtmp://192.168.1.100/live<br/>...vhost...demo.srs.com/livestream</td>
<th>无</th>
<td>查看192.168.1.100上的流</td>
</tr>
</tbody>
</table>

访问其他服务器的流也类似。

## FMLE的奇怪URL方式

FMLE推流时，URL那个地方，有三个可以输入的框，参考[Adobe FMLE](http://help.adobe.com/en_US/FlashMediaLiveEncoder/3.0/Using/WS5b3ccc516d4fbf351e63e3d11c104ba878-7ff7.html)：
* FMS URL: 需要输入rtmp://host:port/app，例如：rtmp://demo.srs.com/live
* Backup URL: 备份的服务器，格式同FMS URL。若指定了备份服务器，FMLE会同时推送给这两个服务器。
* Stream: 流名称，例如：livestream

实际上是将RTMP URL分成了两部分，stream前面那部分和stream。为何要这么搞？我猜想有以下原因：
* 支持多级app和Stream：我们目前举的例子都是一级app和一级stream，实际上RTMP支持多级app和stream，就像子文件夹，实际上很少用得到。所以SRS的URL都是一个地址，默认最后一个/后面就是stream，前面是app。
* 支持流名称带参数：Adobe的鬼HLS/HDS非常之麻烦，那个地址是个恶心的完全不一致。参考[FMS livepkgr](http://help.adobe.com/en_US/flashmediaserver/devguide/WSd391de4d9c7bd609-52e437a812a3725dfa0-8000.html#WSd391de4d9c7bd609-52e437a812a3725dfa0-7ff5)，例如发布一个rtmp，并切片成HLS：
```bash
FMLE:
FMS URL: rtmp://demo.srs.com/livepkgr
Stream: livestream?adbe-live-event=liveevent

Client:
RTMP:  rtmp://demo.srs.com/livepkgr/livestream
HLS: http://demo.srs.com/hls-live/livepkgr/_definst_/liveevent/livestream.m3u8
HDS: http://demo.srs.com/hds-live/livepkgr/_definst_/liveevent/livestream.f4m
```
没有比这个更恶心的东西了。比较SRS的简洁方案：
```bash
FMLE: 
FMS URL: rtmp://demo.srs.com/livepkgr
Stream: livestream

Client:
RTMP: rtmp://demo.srs.com/livepkgr/livestream
HLS: http://demo.srs.com/livepkgr/livestream.m3u8
HDS: not support yet.
```

既然谈到了RTMP URL中的参数，下一章就说说这个。

## RTMP URL参数

RTMP URL一般是不带参数，类似于http的query，有时候为了特殊的要求，会在RTMP URL中带参数，譬如：
* Vhost：前面讲过，在app后面加参数，可以访问指定服务器的指定Vhost。这个SRS的特殊约定，方便排错。
* FMLE的Stream后面的参数，指定event之类的。SRS不需要这么麻烦，HLS是内置支持，无需这种复杂的配置。Callback也是http的，FMS为了支持服务器端脚本，需要很复杂的配置和复杂的参数，实在是很麻烦的设计。
* token认证：SRS还未实现。在连接服务器时，在app后面指定token（方式和vhost一样），例如rtmp://server/live?vhost=xxx&token=xxx/livestream，服务器可以取出token，进行验证，若验证失败则断开连接，这种是比Refer更高级的防盗链。

app和stream后面带参数，这两者有何区别，为何SRS把参数放在app后面？客户端播放流的as3代码大约是：

```as
// how to play url: rtmp://demo.srs.com/live/livestream
conn = new NetConnection();
conn.connect("rtmp://demo.srs.com/live");

stream = new NetStream(conn);
stream.play("livestream");
```

从RTMP协议的角度来看：
* NetConnection.connect(vhost+app)：这一步会完成握手，connect到vhost，切换到app。类似于登录到vhost后，cd到app这个目录。也就是vhost的验证，都可以在这一步做，也就是指定vhost也是在一步了，所以app后面跟的参数都是和vhost/app相关的。
* NetStream.play(stream)：这一步是播放指定的直播流。所以和stream相关的事件，都可以传递参数，譬如Adobe的event。SRS是没有这些事件的，流启动时，若配置了HLS会自动开始切片。

## SRS的URL规则

SRS只做简化的事情，绝对不把简单的事情搞复杂。

SRS的RTMP URL使用标准的RTMP URL，一般不需要对app和stream加参数，或者更改他们的意义。除了两个地方：
* vhost支持参数访问：为了方便运维访问某台服务器的vhost，不需要设置hosts。不影响普通用户。
* 支持token验证：为了支持token验证，在app后面带参数，这个是token验证必须的方式。

另外，SRS建议用户使用一级app和一级stream，不使用多级app和多级stream。譬如：

```bash
// 不推荐使用的多级app或stream
rtmp://demo.srs.com/show/live/livestream
rtmp://demo.srs.com/show/live/livestream/2013
```

srs播放器(srs_player)和srs编码器(srs_publisher)不支持多级app和stream，他们认为最后一个斜杠（/）后面的就是stream，前面的是app。即：

```bash
// srs_player和srs_publisher的解析方式：
// play or publish the following rtmp URL:
rtmp://demo.srs.com/show/live/livestream/2013
schema: rtmp
host/vhost: demo.srs.com
app: show/live/livestream
stream: 2013
```

做此简化的好处是，srs播放器和编码器，只需要指定一个url，而且两者的url是一样的。

SRS常见的三种RTMP URL，详细见下表：
<table>
<thead>
<tr>
<th>URL</th>
<th>说明</th>
</tr>
</thead>
<tbody>
<tr>
<td>rtmp://demo.srs.com/live/livestream</td>
<td>普通用户的标准访问方式，观看直播流</td>
</tr>
<tr>
<td>rtmp://192.168.1.10/live?vhost=demo.srs.com/livestream</td>
<td>运维对特定服务器排错</td>
</tr>
<tr>
<td>rtmp://demo.srs.com/live?key=ER892ID839KD9D0A1D87D/livestream</td>
<td>token验证用户，或者带宽测试的key验证</td>
</tr>
</tbody>
</table>

## SRS的Vhost

SRS的full.conf配置文件中，有很多Vhost，主要是为了说明各个功能，每个功能都单独列出一个vhost。所有功能都放在demo.srs.com这个vhost中。

<table>
<thead>
<th>Category</th>
<th>Vhost</th>
<th>说明</th>
</thead>
<tbody>
<tr>
<td>RTMP</td><td>__defaultVhost__</td><td>默认Vhost的配置，只支持RTMP功能</td>
</tr>
<tr>
<td>RTMP</td><td>chunksize.vhost.com</td><td>如何设置chunk size的实例。其他Vhost将此配置打开，即可设置chunk size。</td>
</tr>
<tr>
<td>Forward</td><td>same.vhost.forward.vhost.com</td><td>Forward实例：将流转发到同一个vhost。</td>
</tr>
<tr>
<td>Forward</td><td>change.vhost.forward.vhost.com</td><td>Forward实例：转发时，转发到服务器的不同vhost。</td>
</tr>
<tr>
<td>HLS</td><td>with-hls.vhost.com</td><td>HLS实例：如何开启HLS，以及HLS的相关配置。</td>
</tr>
<tr>
<td>HLS</td><td>no-hls.vhost.com</td><td>HLS实例：如何禁用HLS。</td>
</tr>
<tr>
<td>RTMP</td><td>min.delay.com</td><td>RTMP最低延迟：如何配置最低延迟的RTMP流。</td>
</tr>
<tr>
<td>RTMP</td><td>refer.anti_suck.com</td><td>Refer实例：如何配置Refer防盗链。</td>
</tr>
<tr>
<td>RTMP</td><td>bandcheck.srs.com</td><td>带宽测试用的vhost，srs测速默认连接到这个vhost。这个vhost配置了带宽测速的key，可测速间隔和最大测速带宽限制。其他Vhost也可以支持测速，只要把这个配置项打开，然后在测速播放器的参数中指明另外的vhost</td>
</tr>
<tr>
<td>RTMP</td><td>removed.vhost.com</td><td>禁用vhost实例：如何禁用vhost。</td>
</tr>
<tr>
<td>Callback</td><td>hooks.callback.vhost.com</td><td>设置http callback的实例，当这些事件发生时，SRS会调用指定的http api。其他Vhost将这些配置打开，就可以支持http callback。</td>
</tr>
<tr>
<td>Transcode</td><td>mirror.transcode.vhost.com</td><td>转码实例：使用ffmpeg的实例filter，将视频做镜像翻转处理。其他Vhost添加这个配置，就可以对流进行转码。<br/>注：所有转码的流都需要重新推送到SRS，使用不同的流名称（vhost和app也可以不一样）。</td>
</tr>
<tr>
<td>Transcode</td><td>drawtext.transcode.vhost.com</td><td>转码实例：在视频上加文字filter。其他Vhost添加此配置，就可以做文字水印。<br/>注：所有转码的流都需要重新推送到SRS，使用不同的流名称（vhost和app也可以不一样）。</td>
</tr>
<tr>
<td>Transcode</td><td>crop.transcode.vhost.com</td><td>转码实例：剪裁视频filter。其他vhost添加此filter，即可对视频进行剪裁。<br/>注：所有转码的流都需要重新推送到SRS，使用不同的流名称（vhost和app也可以不一样）。</td>
</tr>
<tr>
<td>Transcode</td><td>logo.transcode.vhost.com</td><td>转码实例：添加图片/视频水印。其他vhost添加这些配置，可以加图片/视频水印。<br/>注：所有转码的流都需要重新推送到SRS，使用不同的流名称（vhost和app也可以不一样）。</td>
</tr>
<tr>
<td>Transcode</td><td>audio.transcode.vhost.com</td><td>转码实例：只对音频转码。其他vhost添加此配置，可只对音频转码。<br/>注：所有转码的流都需要重新推送到SRS，使用不同的流名称（vhost和app也可以不一样）。</td>
</tr>
<tr>
<td>Transcode</td><td>copy.transcode.vhost.com</td><td>转码实例：只转封装。类似于forward功能。</td>
</tr>
<tr>
<td>Transcode</td><td>all.transcode.vhost.com</td><td>转码实例：对上面的实例的汇总。</td>
</tr>
<tr>
<td>Transcode</td><td>ffempty.transcode.vhost.com</td><td>调用ffempty程序转码，这个只是一个stub，打印出参数而已。用作调试用，看参数是否传递正确。</td>
</tr>
<tr>
<td>Transcode</td><td>app.transcode.vhost.com</td><td>转码实例：只对匹配的app的流进行转码。</td>
</tr>
<tr>
<td>Transcode</td><td>stream.transcode.vhost.com</td><td>转码实例：只对匹配的流进行转码。</td>
</tr>
</tbody>
</table>

SRS的demo.conf配置文件中，包含了demo用到的一些vhost，参考[Usage: Demo](https://github.com/winlinvip/simple-rtmp-server/wiki/v1_CN_SampleDemo)。

<table>
<thead>
<th>Category</th>
<th>Vhost</th>
<th>说明</th>
</thead>
<tbody>
<tr>
<td>DEMO</td><td>players</td><td>srs_player播放的演示流，按照Readme的Step会推流到这个vhost，demo页面打开后播放的流就是这个vhost中的流</td>
</tr>
<tr>
<td>DEMO</td><td>players_pub</td><td>srs编码器推流到players这个vhost，然后转码后将流推送到这个vhost，并切片为hls，srs编码器播放的带字幕的流就是这个vhost的流</td>
</tr>
<tr>
<td>DEMO</td><td>players_pub_rtmp</td><td>srs编码器演示页面中的低延时播放器，播放的就是这个vhost的流，这个vhost关闭了gop cache，关闭了hls，让延时最低（在1秒内）</td>
</tr>
<tr>
<td>DEMO</td><td>demo.srs.com</td><td>srs的演示vhost，Readme的step最后的12路流演示，以及播放器的12路流延时，都是访问的这个vhost。包含了SRS所有的功能。</td>
</tr>
<tr>
<td>Others</td><td>dev</td><td>开发用的，可忽略</td>
</tr>
</tbody>
</table>

Winlin 2014.2