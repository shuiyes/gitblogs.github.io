# WebQQ 协议分析

## （一）：前言

### 项目地址

[ScienJus/qqbot Ruby](https://github.com/ScienJus/qqbot)
[sjdy521/Mojo-Webqq Perl](https://github.com/sjdy521/Mojo-Webqq/)
[b3log/xiaov Java](https://github.com/b3log/xiaov)

### 对Web QQ协议的一些看法

- 整个鉴权模块Cookie、Session、Token全都用上了，真是折腾人
- 加密函数压缩混淆后放在JS还是会被找出来，不过只要偶尔换换也很折腾人
- 整个登录流程分了5步，偶尔加一步、减一步或者改一步同样折腾人（我在写这个项目的时候，正好登录流程就变了，当时还顺便给刚才提到的轮子的作者发了封邮件告诉他这个地方改了）
- 返回数据只提供了返回码而没有描述，因为什么失败了自己猜去吧
- 大部分请求都同时检查了Cookie、Referer、User-Agent和Origin（只有Post请求有）
- 对一些不合理的情况进行了判断，比如登录后却没有发起接收消息的请求
- 对访问频率进行了控制，太频繁的请求也会被拒绝

综上所述，作为一个Web应用，想要不被开发者抓包并模拟请求是很难的。你唯一能做的就是去尽量的恶心他们。并且利用Web项目迭代方便的特性，频繁的进行鉴权等模块的修改，淘汰掉那些开发者没有足够精力维护的项目。

### 写本文的目的

接下来我将介绍一下当前（2015年12月）版本的Web QQ协议，数据来源基本上都是通过Chrome控制台对Smart QQ进行抓包得到的，如果你也想自己实现一个这样的轮子，这些资料可以避免你走很多弯路。

不过在此之前先说一些公共约定：
- 保证请求头中包含正常的User-Agent、Referer、Cookie等信息，如果是Post请求需要额外加上Origin
- 在大部分情况下（除了获取二维码和确认二维码状态），返回内容均为JSON，其中retcode为请求结果（0为成功），response为返回数据
- 不过还有个特例是发送消息的接口，成功时返回的字段是errCode，失败时才是retcode
- 请求失败后，返回的错误码如果是1000000或1000001，几乎可以认为是缺少了第一条中的某个数据
- 如果请求参数中有t，当前版本不会检验它的值，所以我统一设为0.1，但是实际上它的值一般情况下均为当前时间的unix timestamp
- 如果返回的返回的错误码为1000003，很有可能是你的请求频率过于频繁
了解了以上信息后，便可以开始愉快地写接口了


## （二）：登录

自2015年10月起，Web QQ废除了原先的用户名/密码登录，取而代之的是手机QQ扫二维码的登录方式，这也使得Github上几乎所有的相关项目都完全无法使用了。

目前版本的Web QQ协议，整个登录流程包含以下五步：

- 获取二维码
- 确认二维码已被扫描
- 获取鉴权参数ptwebqq
- 获取鉴权参数vfwebqq
- 获取鉴权参数uin和psessionid

登录的目的是获得以下五个参数，用于之后请求其它接口：

* ptwebqq：保存在Cookie中的鉴权信息
* vfwebqq：类似于Token的鉴权信息
* psessionid：类似于SessionId的鉴权信息
* clientid：设备id，为固定值53999199
* uin：登录用户id（其实就是当前登录的QQ号）

### 流程一：获取二维码

请求方式：Get

url：[https://ssl.ptlogin2.qq.com/ptqrshow?appid=501004106&e=0&l=M&s=5&d=72&v=4&t=0.1]()

返回内容：二维码图片（PNG格式）

### 流程二：获取二维码扫描状态

请求方式：Get

url：[https://ssl.ptlogin2.qq.com/ptqrlogin?webqq_type=10&remember_uin=1&login2qq=1&aid=501004106 &u1=http%3A%2F%2Fw.qq.com%2Fproxy.html%3Flogin2qq%3D1%26webqq_type%3D10 &ptredirect=0&ptlang=2052&daid=164&from_ui=1&pttype=1&dumy=&fp=loginerroralert &action=0-0-157510&mibao_css=m_webqq&t=1&g=1&js_type=0&js_ver=10143&login_sig=&pt_randsalt=0]()

referer：[https://ui.ptlogin2.qq.com/cgi-bin/login?daid=164&target=self&style=16&mibao_css=m_webqq&appid=501004106&enable_qlogin=0&no_verifyimg=1 &s_url=http%3A%2F%2Fw.qq.com%2Fproxy.html&f_url=loginerroralert &strong_login=1&login_state=10&t=20131024001]()

返回内容：

扫描前（未失效）：

```
ptuiCB('66','0','','0','二维码未失效。（3203423232）','');
```

扫描前（已失效）：

```
ptuiCB('65','0','','0','二维码已失效。(4012918406)', '');
```

扫描后，认证前：

```
ptuiCB('66','0','','0','二维码认证中。（3203423232）','');
```

认证后：

```
ptuiCB('66','0','','0','http://ptlogin4.web2.qq.com/check_sig?xxxxxx','');
```

这个请求可以直接轮训请求，直到认证成功后，将返回的地址保存下来用作下次请求。

### 流程三：获取ptwebqq

请求方式：Get

url：刚才获得的地址

referer：[http://s.web2.qq.com/proxy.html?v=20130916001&callback=1&id=1]()

这个请求成功后的Http状态码是302，之后需要将Cookie中的ptwebqq保存下来。

### 流程四：获取vfwebqq

请求方式：Get

url：[http://s.web2.qq.com/api/getvfwebqq?ptwebqq=#{ptwebqq}&clientid=53999199&psessionid=&t=0.1]()

referer：[http://s.web2.qq.com/proxy.html?v=20130916001&callback=1&id=1]()

url中需要填入上一步获取到的ptwebqq，请求成功后会返回一个JSON，将result.vfwebqq保存下来。

### 流程五：获取psessionid和uin

请求方式：Post

url：[http://d1.web2.qq.com/channel/login2]()

referer：[http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2]()

表单数据只有一个，Key为r，Value是一个JSON，内容为：

```
{
  "ptwebqq": "#{ptwebqq}",
  "clientid": 53999199,
  "psessionid": "",
  "status": "online",
}
```

其中真正的动态参数只有ptwebqq，请求成功后会返回一个JSON，将result.uin和result.psessionid保存下来。

需要注意的是，这里也返回了一个result.vfwebqq，但是这个值是无用的。

到此为止整个登录流程就结束了，概括一下最开始提到的五个数据：

* ptwebqq：在流程三中通过从Cookie中获得
* vfwebqq：在流程四中通过返回的JSON中获得（在流程五中也会返回一个，不要使用那个）
* uin：在流程五中通过返回的JSON中获得（其实就是qq号）
* psessionid：在流程五中通过返回的JSON中获得
* clientid：固定值为53999199

登录成功后一般可以维持2天左右，并且如今Web QQ是不会出现顶号情况的，但是多终端登录后在接收消息等接口可能会出现冲突的情况


## （三）：收发消息

### 轮训消息

请求方式：Post

url：[http://d1.web2.qq.com/channel/poll2]()

referer：[http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2]()

请求参数只有一个r，值是一个JSON，内容为：

```
{
    "ptwebqq": "#{ptwebqq}",
    "clientid": 53999199,
    "psessionid": "#{psessionid}",
    "key": ""
}
```

ptwebqq和psessionid都是登录后获得的参数。

请求成功后返回的内容为:

```
{
    "result": [
        {
            "poll_type": "message",
            "value": {
                "content": [
                    [
                        "font",
                        {
                            "color": "000000",
                            "name": "微软雅黑",
                            "size": 10,
                            "style": [
                                0,
                                0,
                                0
                            ]
                        }
                    ],
                    "好啊"
                ],
                "from_uin": 3785096088,
                "msg_id": 25477,
                "msg_type": 0,
                "time": 1450686775,
                "to_uin": 931996776
            }
        }
    ],
    "retcode": 0
}
```

poll_type为message表示这是个好友消息。from_uin是用户的编号，可以用于发消息，但不是qq号。to_uin是接受者的编号，同时也是qq号。time为消息的发送时间，content[0]为字体，后面为消息的内容。其他字段暂时不知道有何意义。

如果为群消息，返回内容为：

```
{
    "result": [
        {
            "poll_type": "group_message",
            "value": {
                "content": [
                    [
                        "font",
                        {
                            "color": "000000",
                            "name": "微软雅黑",
                            "size": 10,
                            "style": [
                                0,
                                0,
                                0
                            ]
                        }
                    ],
                    "好啊",
                ],
                "from_uin": 2323421101,
                "group_code": 2323421101,
                "msg_id": 50873,
                "msg_type": 0,
                "send_uin": 3680220215,
                "time": 1450687625,
                "to_uin": 931996776
            }
        }
    ],
    "retcode": 0
}
```

其中poll_type会变成group_message，group_code和from_uin都为群的编号，可以用于发群消息，但不是群号。send_uin为发信息的用户的编号。其他的字段和上面的相同。

如果是讨论组消息，poll_type会变为discu_message，did为讨论组的编号，其他的字段都和群消息相同。

```
{
    "result": [
        {
            "poll_type": "discu_message",
            "value": {
                "content": [
                    [
                        "font",
                        {
                            "color": "000000",
                            "name": "微软雅黑",
                            "size": 10,
                            "style": [
                                0,
                                0,
                                0
                            ]
                        }
                    ],
                    "好啊",
                ],
                "from_uin": 2322423201,
                "did": 2322423201,
                "msg_id": 50873,
                "msg_type": 0,
                "send_uin": 3680220215,
                "time": 1450687625,
                "to_uin": 931996776
            }
        }
    ],
    "retcode": 0
}
```

这里有几点需要注意：

- 服务端收到这个请求后，如果没有新消息，会一直保持住链接，所以遇到ReadTimeout异常是正常的
- Web QQ无法接受图片、@别人、自定义表情等消息，消息内容只有默认表情和文字
- 如果消息内容为表情，content[1]的内容就不是String类型了，而是一个JSONArray类型，里面有表情的编号
- 所以content的长度有可能大于2，代表着消息的内容为文字和表情的混排，content[1]开始的每一位都是分割后的文字或表情
- 这个请求有时候会返回retcode的值为103，此时需要登录Smart QQ，确认能收到消息后点击设置-退出登录，就会恢复正常了
- 在这里接受到的uin、group_code等并不是固定的，而是会改变的，所以不要长时间保存这些信息，

### 发送消息给好友

请求方式：Post

url：[http://d1.web2.qq.com/channel/send_buddy_msg2]()

referer：[http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2]()

请求参数只有一个r，值是一个JSON，内容为：

```
{
    "to": #{user_id},
    "content": [
        "#{msg}",
        [
            "font",
            {
                "name": "宋体",
                "size": 10,
                "style": [
                    0,
                    0,
                    0
                ],
                "color": "000000"
            }
        ]
    ].to_string(),
    "face": 522,
    "clientid": 53999199,
    "msg_id": 65890001,
    "psessionid": "#{psessionid}",
}
```

psessionid是登录后获取的参数，msg是你需要发送的内容，to是用户编号，msg_id只要是一个比较大的数字即可，face暂时不知道有什么用。

这里有一点需要注意的是，我在content的值后面加了一个to_string，因为它不是一个JSONArray类型，而是String类型，在Smart QQ上抓个包也可以发现。如果这里直接提交了一个JSONArray的话，应该会返回1000001错误。

如果发送成功，会返回如下数据：

```
{
    "errCode": 0,
    "msg": "send ok"
}
```

### 发送群消息

请求方式：Post

url：[http://d1.web2.qq.com/channel/send_qun_msg2]()

referer：[http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2]()

请求参数和上面几乎一样，只是将to替换成了group_uin：

```
{
    "group_uin": #{group_uin},
    "content": [
        "#{msg}",
        [
            "font",
            {
                "name": "宋体",
                "size": 10,
                "style": [
                    0,
                    0,
                    0
                ],
                "color": "000000"
            }
        ]
    ].to_string(),
    "face": 522,
    "clientid": 53999199,
    "msg_id": 65890001,
    "psessionid": "#{psessionid}",
}
```

### 发送讨论组消息

请求方式：Post

url：[http://d1.web2.qq.com/channel/send_discu_msg2]()

referer：[http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2]()

格式也是一样的，只是替换为了did：

```
{
    "did": #{discuss_id},
    "content": [
        "#{msg}",
        [
            "font",
            {
                "name": "宋体",
                "size": 10,
                "style": [
                    0,
                    0,
                    0
                ],
                "color": "000000"
            }
        ]
    ].to_string(),
    "face": 522,
    "clientid": 53999199,
    "msg_id": 65890001,
    "psessionid": "#{psessionid}",
}
```


## （四）：好友相关

### 获取好友列表

请求方式：Post

url：[http://s.web2.qq.com/api/get_user_friends2]()

referer：[http://s.web2.qq.com/proxy.html?v=20130916001&callback=1&id=1]()

请求参数只有一个r，值是JSON，内容为：

```
{
    "vfwebqq": "${vfwebqq}",
    "hash": "${hash}"
}
```

vfwebqq依旧是登录后获得的参数，hash是uin和ptwebqq进行加密后的数据，最新的加密算法如下：

```
def self.hash(uin, ptwebqq)
  n = Array.new(4, 0)
  ptwebqq.chars.each_index { |i| n[i % 4] ^= ptwebqq[i].ord }
  u = ['EC', 'OK']
  v = Array.new(4)
  v[0] = uin >> 24 & 255 ^ u[0][0].ord;
  v[1] = uin >> 16 & 255 ^ u[0][1].ord;
  v[2] = uin >> 8 & 255 ^ u[1][0].ord;
  v[3] = uin & 255 ^ u[1][1].ord;
  u = Array.new(8)
  (0...8).each { |i| u[i] = i.odd? ? v[i >> 1] : n[i >> 1] }
  n = ['0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F']
  v = ''
  u.each do |i|
    v << n[(i >> 4) & 15]
    v << n[i & 15]
  end
  v
end
```

perl 语言版
``` perl

sub hash {
my $self = shift;
my $ptwebqq = shift;
my $uin = shift;
$uin .= "";
my @ptb;
for(my $i =0;$i<length($ptwebqq);$i++){
$ptb[$i % 4] ^= ord(substr($ptwebqq,$i,1));
}
my @salt = ("EC", "OK");
my @uinByte;
$uinByte[0] = $uin >> 24 & 0xFF ^ ord(substr($salt[0],0,1));
$uinByte[1] = $uin >> 16 & 0xFF ^ ord(substr($salt[0],1,1));
$uinByte[2] = $uin >> 8 & 0xFF ^ ord(substr($salt[1],0,1));
$uinByte[3] = $uin & 0xFF ^ ord(substr($salt[1],1,1));
my @result;
for(my $i=0;$i<8;$i++){
$result[$i] = $i%2==0?$ptb[$i>>1]:$uinByte[$i>>1];
}
my @hex = ("0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "A", "B", "C", "D", "E", "F");
my $buf = "";
for(my $i=0;$i<@result;$i++){
$buf .= $hex[$result[$i] >> 4 & 0xF];
$buf .= $hex[$result[$i] & 0xF];
}
return $buf;
}
```

java 版
``` java
private static String hash(long x, String K) {
    int[] N = new int[4];
    for (int T = 0; T < K.length(); T++) {
        N[T % 4] ^= K.charAt(T);
    }
    String[] U = {"EC", "OK"};
    long[] V = new long[4];
    V[0] = x >> 24 & 255 ^ U[0].charAt(0);
    V[1] = x >> 16 & 255 ^ U[0].charAt(1);
    V[2] = x >> 8 & 255 ^ U[1].charAt(0);
    V[3] = x & 255 ^ U[1].charAt(1);

    long[] U1 = new long[8];

    for (int T = 0; T < 8; T++) {
        U1[T] = T % 2 == 0 ? N[T >> 1] : V[T >> 1];
    }

    String[] N1 = {"0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "A", "B", "C", "D", "E", "F"};
    String V1 = "";
    for (long aU1 : U1) {
        V1 += N1[(int) ((aU1 >> 4) & 15)];
        V1 += N1[(int) (aU1 & 15)];
    }
    return V1;
}
```

php 版
``` php
function qqhash($uin='', $ptwebqq=''){

    $ptb = array();
    for ($i = 0; $i < strlen($ptwebqq); $i++) {
    	$ptb[$i % 4] ^= ord($ptwebqq[$i]);
    }
    
    $salt = array("EC", "OK");
    $uinByte = array();
    $uinByte[0] = $uin >> 24 & 255 ^ ord($salt[0][0]);
    $uinByte[1] = $uin >> 16 & 255 ^ ord($salt[0][1]);
    $uinByte[2] = $uin >> 8  & 255 ^ ord($salt[1][0]);
    $uinByte[3] = $uin       & 255 ^ ord($salt[1][1]);
    
    $buf = array();
    for ($i = 0; $i < 8; $i++) {
    	$buf[$i] = $i % 2 == 0 ? $ptb[$i >> 1] : $uinByte[$i >> 1];
    }
    
    $hex = array("0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "A", "B", "C", "D", "E", "F");
    $res = "";
    for ($i = 0; $i < 8; $i++) {
    	$res .= $hex[$buf[$i] >> 4 & 15];
    	$res .= $hex[$buf[$i]      & 15];
    }
    
    return $res;
}
```

请求成功后会返回一个如下格式的JSON：

```
{
    "retcode": 0,
    "result": {
        "friends": [
            {
                "flag": 4,
                "uin": 41837138855,
                "categories": 1
            }
        ],
        "marknames": [
            {
                "uin": 37761915054,
                "markname": "备注",
                "type": 0
            }
        ],
        "categories": [
            {
                "index": 0,
                "sort": 2,
                "name": "同学"
            }
        ],
        "vipinfo": [
            {
                "vip_level": 0,
                "u": 20191343597,
                "is_vip": 0
            }
        ],
        "info": [
            {
                "face": 603,
                "flag": 4751942,
                "nick": "昵称",
                "uin": 41837138855
            }
        ]
    }
}
```

其中所有的uin和u都代表着用户的编号。categories保存分组信息，index是编号、sort是顺序、name是名称。marknames的markname是备注，vipinfo的is_vip和vip_level分别代表用户是否为会员和会员等级，info的nick为用户昵称，friends中的categories表示所属分组。

在这里我也很疑惑为什么拆的这么细，解析的时候着实麻烦，最后也只能归结于应该是一个历史遗留问题。

### 获取好友在线状态

刚才刚提到好友列表的JSON拆的很复杂，这里又突然冒出来个单独的接口仅仅是为了获取在线状态。如此设计的原因可能是因为这两个接口的调用频率不同（好友列表很少改变，但是好友的在线状态时时都会改变），或者又是一个历史遗留问题。

请求方式：Get

url：[http://d1.web2.qq.com/channel/get_online_buddies2?vfwebqq=#{vfwebqq}&clientid=53999199&psessionid=#{psessionid}&t=0.1]()

referer：[http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2]()

url中的vfwebqq和psessionid还是一样，就不多说了。请求成功会返回在线的用户，以及终端类型（client_type）：

```
{
    "result": [
        {
            "client_type": 1,
            "status": "online",
            "uin": 3017767504
        }
    ],
    "retcode": 0
}
```

### 通过uin获得QQ号

之前也提到过uin只是一个临时的用户编号，随时都会发生改变。所以如果你想真正知道和你聊天的人是谁，最好还是通过这个接口获得Ta的QQ号。

请求方式：Get

url：[http://s.web2.qq.com/api/get_friend_uin2?tuid=#{uin}&type=1&vfwebqq=#{vfwebqq}&t=0.1]()

referer：[http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2]()

tuid就是对方的uin。请求成功后取出result.account即可：

```
{
    "retcode": 0,
    "result": {
        "uiuin": "",
        "account": 3524125,
        "uin": 1382902354
    }
}
```

### 获取好友详细信息

请求方式：Get

url：[http://s.web2.qq.com/api/get_friend_info2?tuin=#{uin}&vfwebqq=#{vfwebqq}&clientid=53999199&psessionid=#{psessionid}&t=0.1]()

referer：[http://s.web2.qq.com/proxy.html?v=20130916001&callback=1&id=1]()

请求参数没什么特别的就不重复了，请求成功后会返回一堆数据，基本上都是字面上的意思就不多加解释了：

```
{
    "retcode": 0,
    "result": {
        "face": 603,
        "birthday": {
            "month": 8,
            "year": 1895,
            "day": 15
        },
        "occupation": "其他",
        "phone": "110",
        "allow": 1,
        "college": "aaa",
        "uin": 1382902354,
        "constel": 7,
        "blood": 5,
        "homepage": "木有",
        "stat": 20,
        "vip_info": 6,
        "country": "乍得",
        "city": "",
        "personal": "这是简介",
        "nick": "ABCD",
        "shengxiao": 11,
        "email": "352323245@qq.com",
        "province": "",
        "gender": "female",
        "mobile": "139********"
    }
}
```


## （五）：群和讨论组相关

### 获取群列表

请求方式：Post

url：[http://s.web2.qq.com/api/get_group_name_list_mask2]()

referer：[http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2]()

请求参数和获取好友列表一样,请求成功后会返回的JSON格式为：

```
{
    "retcode": 0,
    "result": {
        "gmasklist": [],
        "gnamelist": [
            {
                "flag": 167864331,
                "name": "群名",
                "gid": 14611812014,
                "code": 3157131718
            }
        ],
        "gmarklist": [
            {
                "uin": 18796074161,
                "markname": "备注"
            }
        ]
    }
}
```

gnamelist包含群信息，其中name为群名称，gid为群编号（用于发消息），code也是群编号（用于获取群详细信息），gmarklist包含群的备注信息，markname为备注名。gmasklist目前都是空列表。

### 获取群详细信息

请求方式：Get

url：[http://s.web2.qq.com/api/get_group_info_ext2?gcode=#{group_code}&vfwebqq=#{vfwebqq}&t=0.1]()

referer：[http://s.web2.qq.com/proxy.html?v=20130916001&callback=1&id=1]()

请求参数中的gcode就是群列表中获取的code（注意不是gid）。请求成功后的JSON格式如下：

```
{
    "retcode": 0,
    "result": {
        "stats": [
            {
                "client_type": 1,
                "uin": 2146674552,
                "stat": 50
            }
        ],
        "minfo": [
            {
                "nick": "昵称",
                "province": "北京",
                "gender": "male",
                "uin": 3623536468,
                "country": "中国",
                "city": ""
            }
        ],
        "ginfo": {
            "face": 0,
            "memo": "群公告！",
            "class": 25,
            "fingermemo": "",
            "code": 591539174,
            "createtime": 1231435199,
            "flag": 721421329,
            "level": 4,
            "name": "群名称",
            "gid": 2419762790,
            "owner": 3509557797,
            "members": [
                {
                    "muin": 3623536468,
                    "mflag": 192
                }
            ],
            "option": 2
        },
        "cards": [
            {
                "muin": 3623536468,
                "card": "群名片"
            }
        ],
        "vipinfo": [
            {
                "vip_level": 6,
                "u": 2390929289,
                "is_vip": 1
            }
        ]
    }
}
```

这个请求不光会返回群信息，还会把群里面所有成员的信息都返回，所以数据量会比较大，而且结构复杂解析写起来也麻烦。

其中ginfo包含群的基本信息，memo是群公告、name是群名称、createtime是创建时间、owner是创建者的uin、gid和code分别对应之前群列表的id和code。这里面虽然有一个members表示群成员，但是里面信息太少就不用解析了。stats表示群成员的登录状态，minfo是群成员的基本信息，cards是成员在群中的名片，vipinfo是会员信息，这些在好友列表中都介绍过，就不再重复了。

这里有一点需要注意的是，如果群里没有人改名片，cards并不会返回一个空列表，而是直接没有这个key，所以对于这个字段要进行一下判空操作，否则有可能会出错。

### 获取讨论组列表

请求方式：Get

url：[http://s.web2.qq.com/api/get_discus_list?clientid=53999199&psessionid=#{psessionid}&vfwebqq=#{vfwebqq}&t=0.1]()

referer：[http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2]()

请求成功后遍历dnamelist，did为讨论组编号，name为讨论组名称。

```
{
    "retcode": 0,
    "result": {
        "dnamelist": [
            {
                "did": 167864331,
                "name": "讨论组名",
            }
        ]
    }
}
```

### 获取讨论组详细信息

请求方式：Get

url：[http://d1.web2.qq.com/channel/get_discu_info?did=#{discuss_id}&psessionid=#{psessionid}&vfwebqq=#{vfwebqq}&clientid=53999199&t=0.1]()

referer：[http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2]()

请求成功的JSON格式如下：

```
{
    "result": {
        "info": {
            "did": 236426547,
            "discu_name": "啊啊啊啊",
            "mem_list": [
                {
                    "mem_uin": 3466696377,
                    "ruin": 1301948
                }
            ]
        },
        "mem_info": [
            {
                "nick": "Hey",
                "uin": 3466696377
            }
        ],
        "mem_status": [
            {
                "client_type": 7,
                "status": "online",
                "uin": 3253160543
            }
        ]
    },
    "retcode": 0
}
```

info为讨论组资料，mem_info为讨论组成员的信息，mem_status为登录状态，基本和群信息的格式差不多


## （六）：其他

### 获取最近会话列表

请求方式：Post

url：[http://d1.web2.qq.com/channel/get_recent_list2]()

请求参数依旧只有一个r，值为JSON，内容为：

```
{
  vfwebqq: "#{vfwebqq}",
  clientid: 53999199,
  psessionid: "#{psessionid}"
}
```

这里的psessionid传一个空字符串也是可以的，但是最好还是传了，没准哪一天就校验了呢。

请求成功后会返回一个非常简陋的JSON：

```
{
    "result": [
        {
            "type": 1,
            "uin": 2856977416
        }
    ],
    "retcode": 0
}
```

返回数据只提供了type和uin两个字段，分别表示会话的类型和编号，type的值为0时表示这是一个好友会话，uin就是对方的编号，type为1时表示这是一个群会话，uin为群的编号，type为2时表示是讨论组会话，uin为讨论组编号。

从这里可以明显的看出，Web QQ的服务端倾向于只返回必要的数据，由客户端在本地保存大量的信息和对应的关系。这样做可能主要还是为了性能做打算，但是对接起来真是麻烦爆了，我仅仅是想将消息打印到Console，还要在接收到消息后调用多个接口去查群信息、群成员信息、QQ号等等，实在是麻烦爆了。

### 获取当前登录用户信息

请求方式：Get

url：[http://s.web2.qq.com/api/get_self_info2&t=0.1]()

referer：[http://s.web2.qq.com/proxy.html?v=20130916001&callback=1&id=1]()

返回内容和获取好友详细信息一模一样。

