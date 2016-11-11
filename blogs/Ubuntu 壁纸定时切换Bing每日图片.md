# Ubuntu 壁纸自动切换

## bing.sh

```
#!/bin/bash

while true  
do  
    date +%H:%M:%S
    if [[ $(date +%H:%M:%S) =~ "09:58:" ]] ;then
        rm -f /home/archermind/Pictures/bing.jpg
        /usr/bin/php /usr/local/bing/bing.php
        chmod 777 /home/archermind/Pictures/bing.jpg
        /usr/bin/gsettings 'set' 'org.gnome.desktop.background' 'picture-uri' 'file:///home/archermind/Pictures/bing.jpg'
    fi  
    sleep 60
done
```

## bing.php

```
#!/usr/bin/php -q
<?php

function down($url, $file="", $timeout=60) {
  $file = empty($file) ? pathinfo($url,PATHINFO_BASENAME) : $file;
  //$dir = pathinfo($file,PATHINFO_DIRNAME);
  //!is_dir($dir) && @mkdir($dir,0755,true);
  //$url = str_replace(" ","%20",$url);
  
  if(function_exists('curl_init')) {
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
    $opts = array(
      "http"=>array(
      "method"=>"GET",
      "header"=>"",
      "timeout"=>$timeout)
    );
    $context = stream_context_create($opts);
    if(@copy($url, $file, $context)) {
      //$http_response_header
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
    $filename = "bing.jpg";

    down($url,"/home/archermind/Pictures/".$filename);

    echo $filename."\n";

    //system("/usr/bin/gsettings set org.gnome.desktop.background picture-uri 'file:///home/archermind/Pictures/".$filename."'");
?>
```

## /etc/rc.local
