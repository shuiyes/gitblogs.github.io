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

	if [[ $FLAG == 1 ]];then
		echo "ignore"
    # 每天 9：45 或开机后2分钟(测试用) 设置 Bing 今日图片为壁纸
    elif [[ $(date +%H:%M:%S) =~ "09:45:" || $FLAG == 2 ]];then
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

  // 添加水印
  public function fontMark($x,$y,$font,$color,$text){
    $col = imagecolorallocate($this->image,$color[0],$color[1],$color[2]);
    
    // 无法改变文字大小
    //imagestring($this->image,5,$x,$y,$text,$col);
    imagettftext($this->image, 20, 0, $x,$y, $col , $font, $text);

  }

  // 显示图片or存储
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

// 下载
function down($url, $file="", $timeout=60) {
  $file = empty($file) ? pathinfo($url,PATHINFO_BASENAME) : $file;
  
  if(function_exists('curl_init')) {
    echo "curl download ".$file."\n";
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
    echo "stream_context_create download ".$file."\n";
    $opts = array("http"=>array("method"=>"GET","header"=>"","timeout"=>$timeout));
    $context = stream_context_create($opts);
    if(@copy($url, $file, $context)) {
      return $file;
    } else {
      return false;
    }
  }
}

// 获取图片主色调
function getImageColor($url, $xPos, $yPos){

$isSurportImagick = class_exists('Imagick');
$total = 0;
if($isSurportImagick){
echo "Surport Imagick.\n";
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
    echo "not Surport Imagick.\n";
    $i = imagecreatefromjpeg($url);
    if(!$i) $i = imagecreatefrompng($url);
    
    $rColor = $gColor = $bColor = 0;

    echo 'width: '.imagesx($i).', height: '.imagesy($i).".\n";
    echo '('.$xPos.','.$yPos.','.(imagesx($i)-$xPos).','.($yPos+20).")\n";

    for ($x=$xPos;$x<(imagesx($i)-$xPos);$x++) {
        
        for ($y=$yPos;$y<($yPos+20);$y++) {
            
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

// 颜色取反
function getInverse($color){
    $r = $color['R'];
    $r = $r>100?($r>=100?($r<=155?0:255-$r):255):255-$r;

    $g = $color['G'];
    $g = $g>100?($g>=100?($g<=155?0:255-$g):255):255-$g;

    $b = $color['B'];    
    $b = $b>100?($b>=100?($b<=155?0:255-$b):255):255-$b;

    return array($r,$g,$b);
}

function optim($rgb){
    return ($rgb >= 175) ? 255 : (($rgb <= 80) ? 0 : $rgb);
}

// 175 -> 255, 80 -> 0
function optimize1($color){
    $r = optim($color[0]);
    $g = optim($color[1]);
    $b = optim($color[2]);

    return array($r,$g,$b);
}

// 取rgb 中最大值并置为 255， 使颜色明显
function optimize2($color){
    $r = $color[0];
    $g = $color[1];
    $b = $color[2];

    $find = false;
    if($r == 255 || $g == 255 || $b == 255){
        $find = true;
    }
    if($find){
        if($r != 255) $r = 0;
        if($g != 255) $g = 0;
        if($b != 255) $b = 0;
    }else if($r == 0 && $g == 0 && $b == 0){
        //ignore
    }else{
        if($r > $g){
            if($r>$b){
                $tmp = 0;
                if($r > 120) $tmp = 255;
                $r=$tmp;
                $g=($r-$g<=5)?$tmp:0;
                $b=($r-$b<=5)?$tmp:0;
            }else{
                if($g>$b){
                    $tmp = 0;
                    if($g > 120) $tmp = 255;
                    $r=($g-$r<=5)?$tmp:0;
                    $g=$tmp;
                    $b=($g-$b<=5)?$tmp:0;
                }else{
                    echo 'b';
                    $tmp = 0;
                    if($b > 120) $tmp = 255;
                    $r=($b-$r<=5)?$tmp:0;
                    $g=($b-$g<=5)?$tmp:0;
                    $b=$tmp;
                }
            }
        }else{
            if($g>$b){
                $tmp = 0;
                if($g > 120) $tmp = 255;
                $r=($g-$r<=5)?$tmp:0;
                $g=$tmp;
                $b=($g-$b<=5)?$tmp:0;
            }else{
                if($b>$r){
                    $tmp = 0;
                    if($b > 120) $tmp = 255;
                    $r=($b-$r<=5)?$tmp:0;
                    $g=($b-$g<=5)?$tmp:0;
                    $b=$tmp;
                }else{
                    $tmp = 0;
                    if($r > 120) $tmp = 255;
                    $r=$tmp;
                    $g=($r-$g<=5)?$tmp:0;
                    $b=($r-$b<=5)?$tmp:0;
                }
            }
        }
    }

    return array($r,$g,$b);
}

// 裁剪（裁剪文字颜色区域可知其坐标）
function crop($filename,$x,$y,$w,$h){

    $im = imagecreatefromjpeg($filename); 

    $newim = imagecreatetruecolor($w, $h); 

    imagecopyresampled($newim, $im, 0, 0, $x, $y, $w, $h, $w, $h); 

    $save = '/home/archermind/Pictures/bing/test.jpeg'; 
    ImageJpeg($newim,$save,100); 

    imagedestroy($newim); 
    imagedestroy($im);
}


function getCopyrightTextSize($copyright){
    $textSize = 0;
    for($i = 0; $i < strlen($copyright); $i++){
        
        $ascci = ord($copyright[$i]);
        echo $i.' > '.$ascci."\n";

        if(($ascci > 64 && $ascci < 91) || ($ascci > 96 && $ascci < 123) || $ascci == 47){
            $textSize += 12.5;
        }else{
            $textSize += 25;
        }
    }
    return $textSize;
}

function error(){
    sleep(5);
    system("/usr/bin/php /usr/local/bing/bing.php");
    exit(1);
}

//var_dump($argv);

$day = 0;
if(count($argv) > 1){
    // 参数1 为 天数
    $day = $argv[1];
}else{
    // 没有去随机数
    $day = mt_rand(0,18);
}
echo "day: ".$day."\n";
$bingApi = 'http://cn.bing.com/HPImageArchive.aspx?format=js&n=1&idx='.$day;

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
$path = "/home/archermind/Pictures/bing/";
$file = $path.$filename;

// 文件存在，直接设置壁纸
if(file_exists($file)){
    echo $file." exist, now set to wallpaper.\n";
    system("/usr/bin/gsettings set org.gnome.desktop.background picture-uri file://".$file);
    exit(0);
}
// 不存在，则现下载
else if(down($url, $file)){
    
    // 文字的宽度(模糊值)
    $testSize = strlen($copyright)*11;
    //$testSize = getCopyrightTextSize($copyright);
    $x = (1920 - $testSize)/2;
    $y = 1080-40;

    // 图片主颜色，文字区域的
    $color = getImageColor($file, $x, $y-20);
    echo $color['R']." - ".$color['G']." - ".$color['B']."\n";

    // 颜色取反
    $inverse = getInverse($color);
    /*
    $inverse = array();
    $inverse[0] = $color['R'];
    $inverse[1] = $color['G'];
    $inverse[2] = $color['B'];
    */
    echo $inverse[0]." - ".$inverse[1]." - ".$inverse[2]."\n";

    // 175 -. 255， 80 -> 0
    $inverse = optimize1($inverse);
    echo $inverse[0]." - ".$inverse[1]." - ".$inverse[2]."\n";

    // 取rgb 中最大值并置为 255， 使颜色明显
    $inverse = optimize2($inverse);
    echo $inverse[0]." - ".$inverse[1]." - ".$inverse[2]."\n";

    $obj = new Image_class($file);
    $obj->fontMark($x,$y,$path.'xingshu.ttf',array($inverse[0], $inverse[1], $inverse[2]),$copyright);
    $obj->show(false);

    crop($file,$x,$y-20,$testSize,25);

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
