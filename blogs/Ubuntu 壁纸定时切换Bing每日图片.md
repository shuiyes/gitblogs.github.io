# Ubuntu 壁纸自动切换

## bing.sh

    判断时间开始 Bing每日图片 切换为壁纸

```
#!/bin/bash

FLAG=0

while true 
do
    FLAG=$[$FLAG+1]
    date +%H:%M:%S
    # 每天 9：45 或开机后2分钟(测试用) 设置 Bing 每日图片为壁纸
    if [[ $(date +%H:%M:%S) =~ "09:45:" || $FLAG == 2 ]];then
        echo "now start set bing image to wallpaper."
        /usr/bin/php /usr/local/bing/bing.php
        #break
    fi 
    sleep 60
done
```

## bing.php

    shell 脚本中无法获取网页内容，这里写了各php 脚本来运行。
    包括 Bing api 解析图片地址和图片下载到本地，最后用 gsettings 来设置壁纸

```
#!/usr/bin/php -q
<?php

function down($url, $file="", $timeout=60) {
  $file = empty($file) ? pathinfo($url,PATHINFO_BASENAME) : $file;
  
  if(function_exists('curl_init')) {
    echo "curl download to save ".$file;
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $url);
    curl_setopt($ch, CURLOPT_TIMEOUT, $timeout);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, TRUE);
    $temp = curl_exec($ch);
    if(@file_put_contents($file, $temp) && !curl_error($ch)) {
      return $file;
    } else {
      return false;
    }
  } else {
    $opts = array("http"=>array("method"=>"GET","header"=>"","timeout"=>$timeout));
    $context = stream_context_create($opts);
    if(@copy($url, $file, $context)) {
      return $file;
    } else {
      return false;
    }
  }
}

	$res = file_get_contents('http://cn.bing.com/HPImageArchive.aspx?format=js&idx=0&n=1');
    $arr = json_decode($res);

	$img1 = $arr->{"images"}[0];
    $url = $img1->{"url"};
	$copyright = $img1->{"copyright"};

    $filename = pathinfo($url,PATHINFO_BASENAME);

    down($url,"/home/archermind/Pictures/".$filename);

    echo $filename."\n";
    echo $copyright."\n";

    system("/usr/bin/gsettings set org.gnome.desktop.background picture-uri file:///home/archermind/Pictures/".$filename);
?>
```

## 开机启动爬坑记录

    开机启动自然想到 在 /etc/rc.local 中添加 ，可是运行后测试发现 gsettings 设置无效，分析后发现 root 用户 gsettings 无效(如 sudo /usr/bin/gsettings set org.gnome.desktop.background picture-uri file:///home/archermind/Pictures/xx.jpg)。于是想到以非root 用户运行脚本，如下 su -l xx_user -c bing.sh , 可是测试发现 最后还是无效，各种调试无用。
    最后无奈加到了 ~/.profile 中，注意要设置成后台进程。

```

...

bing.sh > /home/xx/bing.log &

...

```
