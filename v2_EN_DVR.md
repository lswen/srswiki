# DVR

SRS supports DVR RTMP stream to flv file.

## Build

Use `--with-dvr` to enable dvr, while `--without-dvr` to disable it.

For information about the dvr option, read 
[Build](https://github.com/winlinvip/simple-rtmp-server/wiki/v1_EN_Build)

## Config

The difficult of DVR is about the flv name, while SRS use app/stream+random name.
User can use http-callback to rename, for example, when DVR reap flv file.

Config for DVR:

```bash
    # dvr RTMP stream to file,
    # start to record to file when encoder publish,
    # reap flv according by specified dvr_plan.
    dvr {
        # whether enabled dvr features
        # default: off
        enabled         on;
        # the dvr output path.
        # we supports some variables to generate the filename.
        #       [vhost], the vhost of stream.
        #       [app], the app of stream.
        #       [stream], the stream name of stream.
        #       [2006], replace this const to current year.
        #       [01], replace this const to current month.
        #       [02], replace this const to current date.
        #       [15], replace this const to current hour.
        #       [04], repleace this const to current minute.
        #       [05], repleace this const to current second.
        #       [999], repleace this const to current millisecond.
        #       [timestamp],replace this const to current UNIX timestamp in ms.
        # @remark we use golang time format "2006-01-02 15:04:05.999"
        # for example, for url rtmp://ossrs.net/live/livestream and time 2015-01-03 10:57:30.776
        # 1. No variables, the rule of SRS1.0(auto add [stream].[timestamp].flv as filename):
        #       dvr_path ./objs/nginx/html;
        #       =>
        #       dvr_path ./objs/nginx/html/live/livestream.1420254068776.flv;
        # 2. Use stream and date as dir name, time as filename:
        #       dvr_path /data/[vhost]/[app]/[stream]/[2006]/[01]/[02]/[15].[04].[05].[999].flv;
        #       =>
        #       dvr_path /data/ossrs.net/live/livestream/2015/01/03/10.57.30.776.flv;
        # 3. Use stream and year/month as dir name, date and time as filename:
        #       dvr_path /data/[vhost]/[app]/[stream]/[2006]/[01]/[02]-[15].[04].[05].[999].flv;
        #       =>
        #       dvr_path /data/ossrs.net/live/livestream/2015/01/03-10.57.30.776.flv;
        # 4. Use vhost/app and year/month as dir name, stream/date/time as filename:
        #       dvr_path /data/[vhost]/[app]/[2006]/[01]/[stream]-[02]-[15].[04].[05].[999].flv;
        #       =>
        #       dvr_path /data/ossrs.net/live/2015/01/livestream-03-10.57.30.776.flv;
        # @see https://github.com/winlinvip/simple-rtmp-server/wiki/v2_CN_DVR#custom-path
        # @see https://github.com/winlinvip/simple-rtmp-server/wiki/v2_EN_DVR#custom-path
        # default: ./objs/nginx/html
        dvr_path        ./objs/nginx/html;
        # the dvr plan. canbe:
        #   session reap flv when session end(unpublish).
        #   segment reap flv when flv duration exceed the specified dvr_duration.
        # default: session
        dvr_plan        session;
        # the param for plan(segment), in seconds.
        # default: 30
        dvr_duration    30;
        # the param for plan(segment),
        # whether wait keyframe to reap segment,
        # if off, reap segment when duration exceed the dvr_duration,
        # if on, reap segment when duration exceed and got keyframe.
        # default: on
        dvr_wait_keyframe       on;
        # about the stream monotonically increasing:
        #   1. video timestamp is monotonically increasing, 
        #   2. audio timestamp is monotonically increasing,
        #   3. video and audio timestamp is interleaved monotonically increasing.
        # it's specified by RTMP specification, @see 3. Byte Order, Alignment, and Time Format
        # however, some encoder cannot provides this feature, please set this to off to ignore time jitter.
        # the time jitter algorithm:
        #   1. full, to ensure stream start at zero, and ensure stream monotonically increasing.
        #   2. zero, only ensure sttream start at zero, ignore timestamp jitter.
        #   3. off, disable the time jitter algorithm, like atc.
        # default: full
        time_jitter             full;
    }
```

The plan of DVR used to reap flv file:

* session: When start publish, open flv file, close file when unpublish.
* segment: Reap flv file by the dvr_duration and dvr_wait_keyframe.
* time_jitter: The time jitter algorithm to use.
* dvr_path: The path of dvr, the rules is specified at below.

The config file can also use `conf/dvr.segment.conf` or `conf/dvr.session.conf`.

## Custom Path

We can custom the dvr path(dir and filename) by rules:

* Use date and time and stream info as dir name, to avoid too many files in a dir.
* Use date and time and stream info as filename, for better search.
* Provides the data/time and stream info variables, use brackets to identify them.
* Keep SRS1.0 rule, supports write to a specified dir and uses timestamp as filename. If no filename specified(dir specified only), use `[stream].[timestamp].flv` as filename to compatible with SRS1.0 rule.

About the data and time variable, refer to go time format string, for example, use an actual year 2006 instead YYYY, it's a good design:

```
2006-01-02 15:04:05.999
```

The variables of dvr:

1. Year, [2006], replace this const to current year.
1. Month, [01], replace this const to current month.
1. Date, [02], replace this const to current date.
1. Hour, [15], replace this const to current hour.
1. Minute, [04], repleace this const to current minute.
1. Second, [05], repleace this const to current second.
1. Millisecond, [999], repleace this const to current millisecond.
1. Timestamp, [timestamp],replace this const to current UNIX timestamp in ms.
1. Stream info, refer to transcode output, variables are [vhost], [app], [stream]

For example, for url `rtmp://ossrs.net/live/livestream` and time `2015-01-03 10:57:30.776`:

1. No variables, the rule of SRS1.0(auto add `[stream].[timestamp].flv` as filename):
    * dvr_path ./objs/nginx/html;
    * =>
    * dvr_path ./objs/nginx/html/live/livestream.1420254068776.flv;

1. Use stream and date as dir name, time as filename:
    * dvr_path /data/[vhost]/[app]/[stream]/[2006]/[01]/[02]/[15].[04].[05].[999].flv;
    * =>
    * dvr_path /data/ossrs.net/live/livestream/2015/01/03/10.57.30.776.flv;

1. Use stream and year/month as dir name, date and time as filename:
    * dvr_path /data/[vhost]/[app]/[stream]/[2006]/[01]/[02]-[15].[04].[05].[999].flv;
    * =>
    * dvr_path /data/ossrs.net/live/livestream/2015/01/03-10.57.30.776.flv;

1. Use vhost/app and year/month as dir name, stream/date/time as filename:
    * dvr_path /data/[vhost]/[app]/[2006]/[01]/[stream]-[02]-[15].[04].[05].[999].flv;
    * =>
    * dvr_path /data/ossrs.net/live/2015/01/livestream-03-10.57.30.776.flv;

1. Use app as dirname, stream and timestamp as filename(the SRS1.0 rule):
    * dvr_path /data/[app]/[stream].[timestamp].flv;
    * =>
    * dvr_path /data/live/livestream.1420254068776.flv;

## Bug

The bugs of dvr:

* The dir and filename rules: [#179](https://github.com/winlinvip/simple-rtmp-server/issues/179)
* The http callback for dvr: [#274](https://github.com/winlinvip/simple-rtmp-server/issues/274)

## Reload

The changing of dvr and reload will restart the dvr, that is, to close current dvr file then apply new config.

Winlin 2015.1