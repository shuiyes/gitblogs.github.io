# Ubuntu zip 中文乱码

由于zip格式中并没有指定编码格式，Windows下生成的zip文件中的编码是GBK/GB2312等，因此，导致这些zip文件在Linux下解压时出现乱码问题，因为Linux下的默认编码是UTF8。
目前网上流传一种unzip -O cp936的方法，但一些unzip是没有-O这个选项的。
我使用的版本 unzip 6.0 debian modified 版本有这个选项
我发现另外两种解决方案可用。
## python方案
此方案目前来看非常完美。

```
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import os
import sys
import zipfile

#print "Processing File " + sys.argv[1]

file=zipfile.ZipFile(sys.argv[1],"r");
for name in file.namelist():
    utf8name=name.decode('gbk')
#    print "Extracting " + utf8name
    pathname = os.path.dirname(utf8name)
    if not os.path.exists(pathname) and pathname!= "":
        os.makedirs(pathname)
    data = file.read(name)
    if not os.path.exists(utf8name):
        fo = open(utf8name, "w")
        fo.write(data)
        fo.close
file.close()
```

Windows 用户屏蔽两条 print 语句，Linux 用户不用屏蔽

## 7z方案
需要安装p7zip和convmv，在Fedora下的命令是

```
su -c 'yum install p7zip convmv'
```

在ubuntu下的安装命令是
```
sudo apt-get install p7zip convmv
```

安装完之后，就可以用7za和convmv两个命令完成解压缩任务。

```
LANG=C 7za x your-zip-file.zip
convmv -f GBK -t utf8 --notest -r .
```

第一条命令用于解压缩，而LANG=C表示以US-ASCII这样的编码输出文件名，如果没有这个语言设置，它同样会输出乱码，只不过是UTF8格式的乱码(convmv会忽略这样的乱码)。
第二条命令是将GBK编码的文件名转化为UTF8编码，-r表示递归访问目录，即对当前目录中所有文件进行转换。
