# Ubuntu 壁纸自动切换

## bing.sh

    判断时间开始 Bing每日图片 切换为壁纸

```
#!/bin/bash

FLAG=0

D=$(date +%M)
echo $D

while true
do
    FLAG=$[$FLAG+1]

    date +%H:%M:%S
    # 每天 9：45 或开机后2分钟(测试用) 设置 Bing 今日图片为壁纸
    if [[ $(date +%H:%M:%S) =~ "09:45:" || $FLAG == 2 ]];then
        echo "now start set bing image to wallpaper."
        /usr/bin/php /usr/local/bing/bing.php 0
        #break
    # 而后每小时 随机设置 Bing 每日图片为壁纸
    elif [[ $(date +%M) == $D ]];then
        echo "hour start set bing image to wallpaper."
        /usr/bin/php /usr/local/bing/bing.php
    fi
    sleep 60
done
```

## bing.php

    shell 脚本中无法获取网页内容，这里写了各php 脚本来运行。
    包括 Bing api 解析图片地址和图片下载到本地，并在图片底部添加文字 copyright（文字颜色取自图片主色调的反色），最后用 gsettings 来设置壁纸

```
#!/usr/bin/php -q
<?php

class Image_class {
  private $image;
  private $info;
  private $path;
 
  function __construct($src){
    $info = getimagesize($src);
    $type = image_type_to_extension($info[2],false);
    $this -> info =$info;
    $this->info['type'] = $type;
    $fun = "imagecreatefrom" .$type;
    $this -> image = $fun($src);
    $this->path = $src;
  }

  public function fontMark($x,$y,$font,$color,$text){
    $col = imagecolorallocate($this->image,$color[0],$color[1],$color[2]);
    
    // 无法改变文字大小
    //imagestring($this->image,5,$x,$y,$text,$col);
    $testSize = strlen($text)*10;
    $x = $x - $testSize/2;
    imagettftext($this->image, 20, 0, $x,$y, $col , $font, $text);

  }

  public function show($show){
    if($show){
        header('content-type:' . $this -> info['mime']);
        $fun='image' . $this->info['type'];
        $fun($this->image);
    }else{
        @imagejpeg($this->image,$this->path);
    }
  }
 
  function __destruct(){
    imagedestroy($this->image);
  }
}

function down($url, $file="", $timeout=60) {
  $file = empty($file) ? pathinfo($url,PATHINFO_BASENAME) : $file;
  
  if(function_exists('curl_init')) {
    echo "curl download to save ".$file."\n";
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
    echo "stream download to save ".$file."\n";
    $opts = array("http"=>array("method"=>"GET","header"=>"","timeout"=>$timeout));
    $context = stream_context_create($opts);
    if(@copy($url, $file, $context)) {
      return $file;
    } else {
      return false;
    }
  }
}

function getImageColor($url){
$isSurportImagick = class_exists('Imagick');
$total = 0;
if($isSurportImagick){

    $average = new Imagick($url);
    $average->quantizeImage( 10, Imagick::COLORSPACE_RGB, 0, false, false );
    $average->uniqueImageColors();
    function GetImagesColor( Imagick $im ){
        $colorarr = array();
        $it = $im->getPixelIterator();
        $it->resetIterator();
        while( $row = $it->getNextIteratorRow() ){
            foreach ( $row as $pixel ){
                $colorarr[] = $pixel->getColor();
            }
        }
        return $colorarr;
    }
    $colorarr = GetImagesColor($average);
    
    $flag = 0;
    
    foreach($colorarr as $val){
        $rColor += $val['r'];
        $gColor += $val['g'];
        $bColor += $val['b'];
        
        $total++;
    }

}else{

	$i = imagecreatefromjpeg($url);
    if(!$i) $i = imagecreatefrompng($url);
    
    $rColor = $gColor = $bColor = 0;
    for ($x=0;$x<imagesx($i);$x++) {
        
        for ($y=0;$y<imagesy($i);$y++) {
            
            $rgb = imagecolorat($i,$x,$y);
            
            $r = ($rgb >> 16) & 0xFF;
            $g = ($rgb >> 8) & 0xFF;
            $b = $rgb & 0xFF;
            
            $rColor += $r;
            $gColor += $g;
            $bColor += $b;
            
            $total++;
        }
    }  
    
}

$r = round($rColor/$total);
$g = round($gColor/$total);
$b = round($bColor/$total);

$res = array();

$res['R'] = $r;
$res['G'] = $g;
$res['B'] = $b;

return $res;
}

function getInverse($color){
    $r = $color['R'];
    $r = $r>100?($r>=120?($r<150?0:255-$r):255):255-$r;
    $g = $color['G'];
    $g = $g>100?($g>=120?($g<150?0:255-$g):255):255-$g;
    $b = $color['B'];
    $b = $b>100?($b>=120?($b<150?0:255-$b):255):255-$b;
    return array($r,$g,$b);
}

function error(){
    sleep(5);
    system("/usr/bin/php /usr/local/bing/bing.php");
    exit(1);
}

//var_dump($argv);

$day = 0;
if(count($argv) > 1){
    $day = $argv[1];
}else{
    $day = mt_rand(0,18);
}
echo "day: ".$day."\n";
$bingApi = 'http://cn.bing.com/HPImageArchive.aspx?format=js&idx='.$day.'&n=1';

$res = file_get_contents($bingApi);

if(!$res){
    echo "access ".$url." error, try again.\n";
    error();
}

$arr = json_decode($res);

$img1 = $arr->{"images"}[0];
$url = $img1->{"url"};
$copyright = $img1->{"copyright"};

$filename = pathinfo($url,PATHINFO_BASENAME);
$path = "/home/archermind/Pictures/";
$file = $path.$filename;

if(file_exists($file)){
    echo $file." exist, now set to wallpaper.\n";
    system("/usr/bin/gsettings set org.gnome.desktop.background picture-uri file://".$file);
    exit(0);
}else if(down($url, $file)){
    $color = getImageColor($file);

    echo $color['R']." - ".$color['G']." - ".$color['B']."\n";
    $inverse = getInverse($color);
    echo $inverse[0]." - ".$inverse[1]." - ".$inverse[2]."\n";
    
    $obj = new Image_class($file);
    $obj->fontMark(1920/2,1080-40,$path.'xingshu.ttf',array($inverse[0], $inverse[1], $inverse[2]),$copyright);
    $obj->show(false);

    echo "added watermark of ".$copyright.", now set to wallpaper.\n";
    system("/usr/bin/gsettings set org.gnome.desktop.background picture-uri file://".$file);
    exit(0);
}else{
    echo "down ".$url." error, try again.\n";
    error(); 
}
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
